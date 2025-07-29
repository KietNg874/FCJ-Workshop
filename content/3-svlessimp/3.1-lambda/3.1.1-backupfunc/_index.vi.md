---
title : "Lambda Backup Function"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 3.1.1 </b> "
---

1. **Điều hướng đến Lambda Console**
   - Truy cập https://console.aws.amazon.com/lambda/
   - Nhấp **"Create function"**

2. **Cấu hình Cơ bản**
   ```
   Function name: enhanced-dynamodb-backup
   Runtime: Python 3.*
   Architecture: x86_64
   ```

3. **Quyền**
   ```
   Execution role: Use an existing role
   Existing role: EnhancedBackupLambdaRole
   ```

![img](/FCJ-Workshop/images/3.svlessimp/lambda1.png)

4. Nhấp **"Create function"**

5. **Cài đặt Nâng cao**
   ```
   Enable X-Ray tracing: ✅ Active
   Memory: 512 MB
   Timeout: 5 minutes
   Environment variables: (cấu hình sau khi tạo)
   ```
![img](/FCJ-Workshop/images/3.svlessimp/lambda2.png)

Bật X-Ray tracing

![img](/FCJ-Workshop/images/3.svlessimp/lambda3.png)

### Thêm Enhanced Backup Code

```python
import json
import boto3
import hashlib
import os
from datetime import datetime
from decimal import Decimal
from botocore.exceptions import ClientError
import time
import random

# Khởi tạo AWS clients
dynamodb = boto3.resource('dynamodb')
s3 = boto3.client('s3')
sns = boto3.client('sns')
cloudwatch = boto3.client('cloudwatch')

def lambda_handler(event, context):
    """Enhanced backup function với validation và error handling"""
    
    backup_bucket = os.environ.get('BACKUP_BUCKET', 'serverless-backup-ntk')
    success_topic = os.environ.get('SUCCESS_TOPIC_ARN')
    failure_topic = os.environ.get('FAILURE_TOPIC_ARN')
    
    tables = event.get('tables', ['app-users', 'app-orders', 'backup-metadata'])
    backup_results = []
    
    try:
        for table_name in tables:
            result = backup_table_with_retry(table_name, backup_bucket)
            backup_results.append(result)
        
        # Xác thực tất cả backups
        validation_results = validate_backups(backup_results, backup_bucket)
        
        # Publish custom metrics
        publish_backup_metrics(backup_results)
        
        # Gửi thông báo thành công
        if success_topic:
            send_notification(success_topic, 'Backup Success', {
                'backup_results': backup_results,
                'validation_results': validation_results
            })
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': 'Enhanced backup hoàn thành thành công',
                'backup_results': backup_results,
                'validation_results': validation_results
            }, default=str)
        }
        
    except Exception as e:
        error_message = f'Enhanced backup thất bại: {str(e)}'
        
        # Gửi thông báo thất bại
        if failure_topic:
            send_notification(failure_topic, 'Backup Failure', {
                'error': error_message,
                'partial_results': backup_results
            })
        
        return {
            'statusCode': 500,
            'body': json.dumps({
                'message': 'Enhanced backup thất bại',
                'error': str(e)
            })
        }

def backup_table_with_retry(table_name, backup_bucket, max_retries=3):
    """Backup table với exponential backoff retry"""
    
    for attempt in range(max_retries):
        try:
            return backup_single_table(table_name, backup_bucket)
        except ClientError as e:
            if attempt == max_retries - 1:
                raise
            # Exponential backoff với jitter
            wait_time = (2 ** attempt) + random.uniform(0, 1)
            time.sleep(wait_time)

def backup_single_table(table_name, backup_bucket):
    """Backup một DynamoDB table duy nhất"""
    
    table = dynamodb.Table(table_name)
    
    # Scan table với pagination
    items = []
    response = table.scan()
    items.extend(response['Items'])
    
    while 'LastEvaluatedKey' in response:
        response = table.scan(ExclusiveStartKey=response['LastEvaluatedKey'])
        items.extend(response['Items'])
    
    # Chuyển đổi Decimal thành float cho JSON serialization
    for item in items:
        for key, value in item.items():
            if isinstance(value, Decimal):
                item[key] = float(value)
    
    # Tính checksum cho tính toàn vẹn
    checksum = calculate_checksum(items)
    
    # Tạo cấu trúc dữ liệu backup
    backup_data = {
        'table_name': table_name,
        'backup_timestamp': datetime.now().isoformat(),
        'item_count': len(items),
        'data': items,
        'checksum': checksum,
        'version': '2.0'
    }
    
    # Upload lên S3 với default encryption
    timestamp = datetime.now().strftime('%Y-%m-%d-%H-%M-%S')
    filename = f'dynamodb-backups/{table_name}-backup-{timestamp}.json'
    
    s3.put_object(
        Bucket=backup_bucket,
        Key=filename,
        Body=json.dumps(backup_data, indent=2, default=str),
        ServerSideEncryption='AES256',
        Metadata={
            'table-name': table_name,
            'backup-version': '2.0',
            'item-count': str(len(items))
        }
    )
    
    return {
        'table': table_name,
        'filename': filename,
        'item_count': len(items),
        'status': 'success',
        'checksum': checksum,
        'timestamp': datetime.now().isoformat()
    }

def calculate_checksum(items):
    """Tính SHA256 checksum cho tính toàn vẹn dữ liệu"""
    content = json.dumps(items, sort_keys=True, default=str)
    return hashlib.sha256(content.encode()).hexdigest()

def validate_backups(backup_results, backup_bucket):
    """Xác thực tính toàn vẹn backup"""
    validation_results = []
    
    for backup in backup_results:
        try:
            # Download và verify backup file
            response = s3.get_object(Bucket=backup_bucket, Key=backup['filename'])
            backup_data = json.loads(response['Body'].read())
            
            # Verify checksum
            calculated_checksum = calculate_checksum(backup_data['data'])
            checksum_valid = calculated_checksum == backup_data['checksum']
            
            validation_results.append({
                'table': backup['table'],
                'filename': backup['filename'],
                'checksum_valid': checksum_valid,
                'item_count_match': backup_data['item_count'] == backup['item_count'],
                'status': 'valid' if checksum_valid else 'invalid'
            })
            
        except Exception as e:
            validation_results.append({
                'table': backup['table'],
                'filename': backup['filename'],
                'status': 'validation_failed',
                'error': str(e)
            })
    
    return validation_results

def publish_backup_metrics(backup_results):
    """Publish custom CloudWatch metrics"""
    
    success_count = sum(1 for r in backup_results if r['status'] == 'success')
    total_count = len(backup_results)
    success_rate = (success_count / total_count) * 100 if total_count > 0 else 0
    
    total_items = sum(r['item_count'] for r in backup_results)
    
    cloudwatch.put_metric_data(
        Namespace='BackupDR/Enhanced',
        MetricData=[
            {
                'MetricName': 'BackupSuccessRate',
                'Value': success_rate,
                'Unit': 'Percent',
                'Dimensions': [
                    {'Name': 'Environment', 'Value': 'Development'}
                ]
            },
            {
                'MetricName': 'TotalItemsBackedUp',
                'Value': total_items,
                'Unit': 'Count'
            },
            {
                'MetricName': 'TablesProcessed',
                'Value': total_count,
                'Unit': 'Count'
            }
        ]
    )

def send_notification(topic_arn, subject, message_data):
    """Gửi SNS notification"""
    sns.publish(
        TopicArn=topic_arn,
        Subject=subject,
        Message=json.dumps(message_data, indent=2, default=str)
    )

```

![img](/FCJ-Workshop/images/3.svlessimp/lambda4.png)

### Cấu hình Environment Variables

1. Truy cập **Configuration** → **Environment variables**
2. Thêm các biến này:
   ```
   BACKUP_BUCKET = serverless-backup-primary-[yourname]
   SUCCESS_TOPIC_ARN = arn:aws:sns:us-east-1:ACCOUNT:backup-success-notifications
   FAILURE_TOPIC_ARN = arn:aws:sns:us-east-1:ACCOUNT:backup-failure-notifications
   ```
   *Thay thế bằng tên bucket thực tế và SNS topic ARNs của bạn*

![img](/FCJ-Workshop/images/3.svlessimp/lambda5.png)
