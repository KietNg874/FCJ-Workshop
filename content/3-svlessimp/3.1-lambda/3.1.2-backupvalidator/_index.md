---
title : "Lambda Backup Validator Function"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 3.1.2 </b> "
---

### Create Backup Validator Function

1. **Create new Lambda function**:
   ```
   Function name: backup-validator
   Runtime: Python 3.*
   Execution role: EnhancedBackupLambdaRole
   Memory: 256 MB
   Timeout: 3 minutes
   ```

![img](/images/3.svlessimp/lambda6.png)

2. **Add validation code**:

```python
import json
import boto3
import hashlib
from datetime import datetime, timezone

s3 = boto3.client('s3')
sns = boto3.client('sns')

def lambda_handler(event, context):
    """Validate backup integrity and completeness"""
    
    backup_bucket = event.get('backup_bucket', 'serverless-backup-ntk')
    backup_files = event.get('backup_files', [])
    
    # Validate input
    if not backup_files:
        return {
            'statusCode': 400,
            'body': json.dumps({
                'message': 'No backup files provided for validation',
                'validation_results': [],
                'overall_success': False
            })
        }
    
    validation_results = []
    
    for backup_file in backup_files:
        try:
            # Download backup file
            filename = backup_file.get('filename') if isinstance(backup_file, dict) else backup_file
            response = s3.get_object(Bucket=backup_bucket, Key=filename)
            backup_data = json.loads(response['Body'].read())
            
            # Perform validation checks
            validation_result = {
                'filename': filename,
                'table': backup_data.get('table_name', 'unknown'),
                'timestamp': datetime.now(timezone.utc).isoformat(),
                'checks': {
                    'file_readable': True,
                    'structure_valid': validate_structure(backup_data),
                    'checksum_valid': validate_checksum(backup_data),
                    'item_count_positive': backup_data.get('item_count', 0) > 0,
                    'timestamp_recent': validate_timestamp(backup_data.get('backup_timestamp', ''))
                }
            }
            
            validation_result['overall_status'] = all(validation_result['checks'].values())
            validation_results.append(validation_result)
            
        except Exception as e:
            validation_results.append({
                'filename': backup_file.get('filename') if isinstance(backup_file, dict) else backup_file,
                'overall_status': False,
                'error': str(e),
                'timestamp': datetime.now(timezone.utc).isoformat(),
                'checks': {
                    'file_readable': False,
                    'structure_valid': False,
                    'checksum_valid': False,
                    'item_count_positive': False,
                    'timestamp_recent': False
                }
            })
    
    overall_success = all(r.get('overall_status', False) for r in validation_results)
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'validation_results': validation_results,
            'overall_success': overall_success,
            'total_files_validated': len(validation_results),
            'successful_validations': sum(1 for r in validation_results if r.get('overall_status', False))
        })
    }

def validate_structure(backup_data):
    """Validate backup data structure"""
    required_fields = ['table_name', 'backup_timestamp', 'item_count', 'data', 'checksum']
    
    if not isinstance(backup_data, dict):
        return False
    
    # Check all required fields exist
    if not all(field in backup_data for field in required_fields):
        return False
    
    # Additional structure validation
    if not isinstance(backup_data.get('data'), list):
        return False
    
    if not isinstance(backup_data.get('item_count'), int):
        return False
    
    return True

def validate_checksum(backup_data):
    """Validate data integrity using checksum"""
    if 'checksum' not in backup_data or 'data' not in backup_data:
        return False
    
    try:
        content = json.dumps(backup_data['data'], sort_keys=True, default=str)
        calculated_checksum = hashlib.sha256(content.encode()).hexdigest()
        return calculated_checksum == backup_data['checksum']
    except Exception:
        return False

def validate_timestamp(timestamp_str):
    """Validate backup timestamp is recent (within 24 hours)"""
    try:
        # Handle different timestamp formats
        if timestamp_str.endswith('Z'):
            backup_time = datetime.fromisoformat(timestamp_str.replace('Z', '+00:00'))
        elif '+' in timestamp_str or timestamp_str.endswith('00:00'):
            backup_time = datetime.fromisoformat(timestamp_str)
        else:
            # Assume UTC if no timezone info
            backup_time = datetime.fromisoformat(timestamp_str).replace(tzinfo=timezone.utc)
        
        current_time = datetime.now(timezone.utc)
        
        # Convert backup_time to UTC if it has timezone info
        if backup_time.tzinfo is not None:
            backup_time_utc = backup_time.astimezone(timezone.utc)
        else:
            backup_time_utc = backup_time.replace(tzinfo=timezone.utc)
        
        time_diff = current_time - backup_time_utc
        return time_diff.total_seconds() < 86400  # 24 hours
        
    except Exception as e:
        print(f"Timestamp validation error: {str(e)}")
        return False

```