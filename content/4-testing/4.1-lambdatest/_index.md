---
title : "Test Lambda"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 4.1. </b> "
---

### Objectives
- Test individual Lambda functions in isolation
- Validate function responses and error handling
- Verify environment variable configuration

### Test Enhanced Backup Function

1. **Navigate to Lambda Console** → `enhanced-dynamodb-backup`
2. **Go to Test tab** → **Create new test event**
3. **Event name**: `comprehensive-backup-test`
4. **Test event**:
```json
{
  "tables": ["app-users", "app-orders", "backup-metadata"]
}
```

5. **Execute test** and verify:
   - ✅ Status: Succeeded
   - ✅ Duration: < 30 seconds
   - ✅ Response contains backup_results
   - ✅ Response contains validation_results

![img](/images/4.testing/TEST_Lambda.png)

6. **Check S3 bucket** for new backup files:
   - Navigate to your primary S3 bucket
   - Check `dynamodb-backups/` folder
   - Verify 3 new backup files created
   - Check file metadata and encryption

![img](/images/4.testing/TEST_s3.png)

7. **Verify SNS notification** received in email

![img](/images/4.testing/TEST_mail.png)

{{% notice tip %}}
**Lambda Testing**: Always test your Lambda functions with realistic data. The test events should mirror what your function will receive in production.
{{% /notice %}}

### Test Backup Validator Function

1. **Navigate to** `backup-validator` **function**
2. **Create test event** with actual backup file:
```json
{
  "backup_bucket": "serverless-backup-primary-[yourname]",
  "backup_files": [
    {
      "filename": "dynamodb-backups/app-users-backup-2025-07-27-XX-XX-XX.json"
    }
  ]
}
```
*Replace with actual filename from your S3 bucket*

3. **Execute test** and verify:
   - ✅ Status: Succeeded
   - ✅ overall_success: true
   - ✅ All validation checks pass

### Test Recovery Orchestrator Function

1. **Navigate to** `enhanced-recovery-orchestrator` **function**
2. **Create test event**:
```json
{
  "table_name": "app-users",
  "backup_bucket": "serverless-backup-primary-[yourname]"
}
```

3. **Execute test** and verify:
   - ✅ Status: Succeeded
   - ✅ Items restored count matches expected
   - ✅ Validation passed: true
