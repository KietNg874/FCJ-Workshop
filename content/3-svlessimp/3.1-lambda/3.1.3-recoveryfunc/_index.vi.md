---
title : "Lambda Enhanced Recovery Function"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3.1.3 </b> "
---

Function này xử lý việc khôi phục dữ liệu từ S3 backups trở lại DynamoDB tables. Nó bao gồm xác thực để đảm bảo khôi phục thành công.

1. **Tạo Lambda function mới**:
   ```
   Function name: enhanced-recovery-orchestrator
   Runtime: Python 3.*
   Execution role: EnhancedBackupLambdaRole
   Memory: 512 MB
   Timeout: 10 minutes
   ```

2. **Thêm recovery code** 

```python
import json
import boto3
import os
from datetime import datetime
from decimal import Decimal

dynamodb = boto3.resource('dynamodb')
s3 = boto3.client('s3')
sns = boto3.client('sns')
cloudwatch = boto3.client('cloudwatch')

def lambda_handler(event, context):
    """Enhanced recovery với validation và monitoring"""
    
    backup_bucket = event.get('backup_bucket', os.environ.get('BACKUP_BUCKET', 'serverless-backup-ntk'))
    table_name = event.get('table_name')
    backup_file = event.get('backup_file')
    batch_size = event.get('batch_size', 25)  # Kích thước batch có thể cấu hình
    
    if not table_name:
        return {
            'statusCode': 400,
            'body': json.dumps({
                'message': 'table_name là bắt buộc',
                'error': 'Thiếu tham số bắt buộc'
            })
        }
    
    try:
        # Tìm backup file nếu không được chỉ định
        if not backup_file:
            backup_file = find_latest_backup(backup_bucket, table_name)
        
        # Download và validate backup
        backup_data = download_and_validate_backup(backup_bucket, backup_file)
        
        # Thực hiện recovery
        recovery_result = perform_table_recovery(table_name, backup_data, batch_size)
        
        # Validate recovery
        validation_result = validate_recovery(table_name, backup_data)
        
        # Publish metrics
        publish_recovery_metrics(recovery_result, validation_result)
        
        # Gửi thông báo
        send_recovery_notification('success', {
            'table_name': table_name,
            'backup_file': backup_file,
            'recovery_result': recovery_result,
            'validation_result': validation_result
        })
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': 'Recovery hoàn thành thành công',
                'table_name': table_name,
                'backup_file': backup_file,
                'items_restored': recovery_result['items_restored'],
                'validation_passed': validation_result['passed']
            })
        }
        
    except Exception as e:
        error_message = f'Recovery thất bại cho table {table_name}: {str(e)}'
        
        send_recovery_notification('failure', {
            'table_name': table_name,
            'backup_file': backup_file,
            'error': error_message
        })
        
        return {
            'statusCode': 500,
            'body': json.dumps({
                'message': 'Recovery thất bại',
                'error': str(e)
            })
        }

def find_latest_backup(bucket, table_name):
    """Tìm backup file gần nhất cho một table"""
    
    response = s3.list_objects_v2(
        Bucket=bucket,
        Prefix=f'dynamodb-backups/{table_name}-backup-'
    )
    
    if 'Contents' not in response:
        raise Exception(f'Không tìm thấy backup files cho table {table_name}')
    
    # Sắp xếp theo last modified và lấy gần nhất
    latest_backup = sorted(response['Contents'], key=lambda x: x['LastModified'])[-1]
    return latest_backup['Key']

def download_and_validate_backup(bucket, backup_file):
    """Download backup file và validate tính toàn vẹn"""
    
    response = s3.get_object(Bucket=bucket, Key=backup_file)
    backup_data = json.loads(response['Body'].read())
    
    # Validate cấu trúc backup
    required_fields = ['table_name', 'backup_timestamp', 'item_count', 'data', 'checksum']
    if not all(field in backup_data for field in required_fields):
        raise Exception('Cấu trúc backup file không hợp lệ')
    
    # Validate checksum
    import hashlib
    content = json.dumps(backup_data['data'], sort_keys=True, default=str)
    calculated_checksum = hashlib.sha256(content.encode()).hexdigest()
    
    if calculated_checksum != backup_data['checksum']:
        raise Exception('Kiểm tra tính toàn vẹn backup file thất bại')
    
    return backup_data

def perform_table_recovery(table_name, backup_data, batch_size=25):
    """Khôi phục dữ liệu vào DynamoDB table"""
    
    table = dynamodb.Table(table_name)
    
    restored_count = 0
    failed_items = []
    
    # Xử lý items theo batches
    items = backup_data['data']
    
    for i in range(0, len(items), batch_size):
        batch = items[i:i + batch_size]
        
        try:
            with table.batch_writer() as batch_writer:
                for item in batch:
                    batch_writer.put_item(Item=item)
                    restored_count += 1
                    
        except Exception as e:
            failed_items.extend([{
                'item': item,
                'error': str(e)
            } for item in batch])
    
    return {
        'items_restored': restored_count,
        'failed_items': len(failed_items),
        'failed_details': failed_items[:10],  # 10 lỗi đầu tiên để debug
        'recovery_timestamp': datetime.now().isoformat()
    }

def get_table_key_schema(table_name):
    """Lấy key schema cho một DynamoDB table"""
    try:
        table = dynamodb.Table(table_name)
        table.load()
        
        key_schema = {}
        for key in table.key_schema:
            key_schema[key['AttributeName']] = key['KeyType']
        
        return key_schema
    except Exception as e:
        print(f"Lỗi khi lấy key schema cho table {table_name}: {str(e)}")
        return {}

def validate_recovery(table_name, backup_data):
    """Validate rằng recovery đã thành công"""
    
    table = dynamodb.Table(table_name)
    
    # Lấy số lượng item hiện tại
    try:
        response = table.describe_table()
        current_count = response['Table']['ItemCount']
    except Exception:
        current_count = 0
    
    expected_count = backup_data['item_count']
    
    # Lấy table key schema để truy xuất item đúng cách
    key_schema = get_table_key_schema(table_name)
    
    # Sample validation - kiểm tra một vài items ngẫu nhiên
    sample_items = backup_data['data'][:min(5, len(backup_data['data']))]  # Kiểm tra 5 items đầu hoặc tất cả nếu ít hơn
    validation_results = []
    
    for item in sample_items:
        try:
            # Trích xuất key attributes dựa trên table schema
            if key_schema:
                key = {attr_name: item[attr_name] for attr_name in key_schema.keys() if attr_name in item}
            else:
                # Fallback: thử các mẫu key phổ biến
                possible_keys = ['id', 'userId', 'orderId', 'backupId', 'pk', 'sk']
                key = {k: v for k, v in item.items() if k in possible_keys}
            
            if not key:
                # Nếu không tìm thấy key, bỏ qua validation cho item này
                validation_results.append(False)
                continue
            
            # Lấy item từ table
            response = table.get_item(Key=key)
            
            if 'Item' in response:
                validation_results.append(True)
            else:
                validation_results.append(False)
                
        except Exception as e:
            print(f"Lỗi validation cho item: {str(e)}")
            validation_results.append(False)
    
    return {
        'passed': all(validation_results) if validation_results else False,
        'current_count': current_count,
        'expected_count': expected_count,
        'sample_validation_rate': sum(validation_results) / len(validation_results) * 100 if validation_results else 0,
        'validation_timestamp': datetime.now().isoformat(),
        'samples_tested': len(validation_results)
    }

def publish_recovery_metrics(recovery_result, validation_result):
    """Publish recovery metrics lên CloudWatch"""
    
    cloudwatch.put_metric_data(
        Namespace='BackupDR/Recovery',
        MetricData=[
            {
                'MetricName': 'ItemsRestored',
                'Value': recovery_result['items_restored'],
                'Unit': 'Count'
            },
            {
                'MetricName': 'RecoveryValidationRate',
                'Value': validation_result['sample_validation_rate'],
                'Unit': 'Percent'
            },
            {
                'MetricName': 'FailedItems',
                'Value': recovery_result['failed_items'],
                'Unit': 'Count'
            }
        ]
    )

def send_recovery_notification(status, data):
    """Gửi thông báo recovery"""
    
    topic_arn = os.environ.get('SUCCESS_TOPIC_ARN' if status == 'success' else 'FAILURE_TOPIC_ARN')
    
    if topic_arn:
        sns.publish(
            TopicArn=topic_arn,
            Subject=f'Recovery {status.title()}',
            Message=json.dumps(data, indent=2, default=str)
        )

```

### Tính năng Chính của Recovery Function

#### **🔍 Tự động Tìm Backup**
- Tìm backup file gần nhất nếu không được chỉ định
- Hỗ trợ chỉ định backup file cụ thể
- Xử lý lỗi khi không tìm thấy backup

#### **✅ Xác thực Backup**
- Kiểm tra cấu trúc file backup
- Xác minh checksum để đảm bảo tính toàn vẹn
- Từ chối khôi phục nếu backup bị hỏng

#### **⚡ Khôi phục Batch**
- Xử lý dữ liệu theo batches để tối ưu hiệu suất
- Kích thước batch có thể cấu hình
- Xử lý lỗi cho từng batch riêng biệt

#### **🔬 Validation sau Khôi phục**
- Kiểm tra số lượng items được khôi phục
- Sample validation để xác minh dữ liệu
- Báo cáo tỷ lệ thành công

#### **📊 Monitoring & Metrics**
- Publish metrics lên CloudWatch
- Theo dõi số items được khôi phục
- Tỷ lệ validation và lỗi

#### **📧 Thông báo**
- Gửi thông báo thành công/thất bại qua SNS
- Chi tiết kết quả khôi phục
- Thông tin lỗi để troubleshooting

### Cách sử dụng

```json
{
  "table_name": "app-users",
  "backup_bucket": "serverless-backup-primary-yourname",
  "backup_file": "dynamodb-backups/app-users-backup-2025-01-28-10-30-00.json",
  "batch_size": 25
}
```

3. **Cấu hình environment variables** (giống như backup function)
