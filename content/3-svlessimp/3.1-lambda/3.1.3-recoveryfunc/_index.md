---
title : "Lambda Enhanced Recovery Function"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3.1.3 </b> "
---

This function handles data restoration from S3 backups back to DynamoDB tables. It includes validation to ensure recovery success.

1. **Create new Lambda function**:
   ```
   Function name: enhanced-recovery-orchestrator
   Runtime: Python 3.*
   Execution role: EnhancedBackupLambdaRole
   Memory: 512 MB
   Timeout: 10 minutes
   ```

2. **Add recovery code** 

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
    """Enhanced recovery with validation and monitoring"""
    
    backup_bucket = event.get('backup_bucket', os.environ.get('BACKUP_BUCKET', 'serverless-backup-ntk'))
    table_name = event.get('table_name')
    backup_file = event.get('backup_file')
    batch_size = event.get('batch_size', 25)  # Configurable batch size
    
    if not table_name:
        return {
            'statusCode': 400,
            'body': json.dumps({
                'message': 'table_name is required',
                'error': 'Missing required parameter'
            })
        }
    
    try:
        # Find backup file if not specified
        if not backup_file:
            backup_file = find_latest_backup(backup_bucket, table_name)
        
        # Download and validate backup
        backup_data = download_and_validate_backup(backup_bucket, backup_file)
        
        # Perform recovery
        recovery_result = perform_table_recovery(table_name, backup_data, batch_size)
        
        # Validate recovery
        validation_result = validate_recovery(table_name, backup_data)
        
        # Publish metrics
        publish_recovery_metrics(recovery_result, validation_result)
        
        # Send notification
        send_recovery_notification('success', {
            'table_name': table_name,
            'backup_file': backup_file,
            'recovery_result': recovery_result,
            'validation_result': validation_result
        })
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': 'Recovery completed successfully',
                'table_name': table_name,
                'backup_file': backup_file,
                'items_restored': recovery_result['items_restored'],
                'validation_passed': validation_result['passed']
            })
        }
        
    except Exception as e:
        error_message = f'Recovery failed for table {table_name}: {str(e)}'
        
        send_recovery_notification('failure', {
            'table_name': table_name,
            'backup_file': backup_file,
            'error': error_message
        })
        
        return {
            'statusCode': 500,
            'body': json.dumps({
                'message': 'Recovery failed',
                'error': str(e)
            })
        }

def find_latest_backup(bucket, table_name):
    """Find the most recent backup file for a table"""
    
    response = s3.list_objects_v2(
        Bucket=bucket,
        Prefix=f'dynamodb-backups/{table_name}-backup-'
    )
    
    if 'Contents' not in response:
        raise Exception(f'No backup files found for table {table_name}')
    
    # Sort by last modified and get the most recent
    latest_backup = sorted(response['Contents'], key=lambda x: x['LastModified'])[-1]
    return latest_backup['Key']

def download_and_validate_backup(bucket, backup_file):
    """Download backup file and validate integrity"""
    
    response = s3.get_object(Bucket=bucket, Key=backup_file)
    backup_data = json.loads(response['Body'].read())
    
    # Validate backup structure
    required_fields = ['table_name', 'backup_timestamp', 'item_count', 'data', 'checksum']
    if not all(field in backup_data for field in required_fields):
        raise Exception('Invalid backup file structure')
    
    # Validate checksum
    import hashlib
    content = json.dumps(backup_data['data'], sort_keys=True, default=str)
    calculated_checksum = hashlib.sha256(content.encode()).hexdigest()
    
    if calculated_checksum != backup_data['checksum']:
        raise Exception('Backup file integrity check failed')
    
    return backup_data

def perform_table_recovery(table_name, backup_data, batch_size=25):
    """Restore data to DynamoDB table"""
    
    table = dynamodb.Table(table_name)
    
    restored_count = 0
    failed_items = []
    
    # Process items in batches
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
        'failed_details': failed_items[:10],  # First 10 failures for debugging
        'recovery_timestamp': datetime.now().isoformat()
    }

def get_table_key_schema(table_name):
    """Get the key schema for a DynamoDB table"""
    try:
        table = dynamodb.Table(table_name)
        table.load()
        
        key_schema = {}
        for key in table.key_schema:
            key_schema[key['AttributeName']] = key['KeyType']
        
        return key_schema
    except Exception as e:
        print(f"Error getting key schema for table {table_name}: {str(e)}")
        return {}

def validate_recovery(table_name, backup_data):
    """Validate that recovery was successful"""
    
    table = dynamodb.Table(table_name)
    
    # Get current item count
    try:
        response = table.describe_table()
        current_count = response['Table']['ItemCount']
    except Exception:
        current_count = 0
    
    expected_count = backup_data['item_count']
    
    # Get table key schema for proper item retrieval
    key_schema = get_table_key_schema(table_name)
    
    # Sample validation - check a few random items
    sample_items = backup_data['data'][:min(5, len(backup_data['data']))]  # Check first 5 items or all if less
    validation_results = []
    
    for item in sample_items:
        try:
            # Extract key attributes based on table schema
            if key_schema:
                key = {attr_name: item[attr_name] for attr_name in key_schema.keys() if attr_name in item}
            else:
                # Fallback: try common key patterns
                possible_keys = ['id', 'userId', 'orderId', 'backupId', 'pk', 'sk']
                key = {k: v for k, v in item.items() if k in possible_keys}
            
            if not key:
                # If no key found, skip validation for this item
                validation_results.append(False)
                continue
            
            # Get item from table
            response = table.get_item(Key=key)
            
            if 'Item' in response:
                validation_results.append(True)
            else:
                validation_results.append(False)
                
        except Exception as e:
            print(f"Validation error for item: {str(e)}")
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
    """Publish recovery metrics to CloudWatch"""
    
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
    """Send recovery notification"""
    
    topic_arn = os.environ.get('SUCCESS_TOPIC_ARN' if status == 'success' else 'FAILURE_TOPIC_ARN')
    
    if topic_arn:
        sns.publish(
            TopicArn=topic_arn,
            Subject=f'Recovery {status.title()}',
            Message=json.dumps(data, indent=2, default=str)
        )

```


3. **Configure environment variables** (same as backup function)
