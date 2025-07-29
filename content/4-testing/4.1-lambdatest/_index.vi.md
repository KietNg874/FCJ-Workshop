---
title : "Test Lambda"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 4.1. </b> "
---

### Test Enhanced Backup Function

1. **Điều hướng đến Lambda Console** → `enhanced-dynamodb-backup`
2. **Truy cập Test tab** → **Create new test event**
3. **Event name**: `comprehensive-backup-test`
4. **Test event**:
```json
{
  "tables": ["app-users", "app-orders", "backup-metadata"]
}
```

5. **Thực thi test** và xác minh:
   - ✅ Status: Succeeded
   - ✅ Duration: < 30 giây
   - ✅ Response chứa backup_results
   - ✅ Response chứa validation_results

![img](/FCJ-Workshop/images/4.testing/TEST_Lambda.png)

6. **Kiểm tra S3 bucket** cho các backup files mới:
   - Điều hướng đến primary S3 bucket của bạn
   - Kiểm tra thư mục `dynamodb-backups/`
   - Xác minh 3 backup files mới được tạo
   - Kiểm tra metadata và encryption của file

![img](/FCJ-Workshop/images/4.testing/TEST_s3.png)

7. **Xác minh SNS notification** nhận được trong email

![img](/FCJ-Workshop/images/4.testing/TEST_mail.png)

{{% notice tip %}}
**Lambda Testing**: Luôn kiểm thử các Lambda functions của bạn với dữ liệu thực tế. Các test events nên phản ánh những gì function của bạn sẽ nhận được trong production.
{{% /notice %}}

### Test Backup Validator Function

1. **Điều hướng đến** `backup-validator` **function**
2. **Tạo test event** với backup file thực tế:
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
*Thay thế bằng filename thực tế từ S3 bucket của bạn*

3. **Thực thi test** và xác minh:
   - ✅ Status: Succeeded
   - ✅ overall_success: true
   - ✅ Tất cả validation checks pass

### Test Recovery Orchestrator Function

1. **Điều hướng đến** `enhanced-recovery-orchestrator` **function**
2. **Tạo test event**:
```json
{
  "table_name": "app-users",
  "backup_bucket": "serverless-backup-primary-[yourname]"
}
```

3. **Thực thi test** và xác minh:
   - ✅ Status: Succeeded
   - ✅ Số lượng Items restored khớp với mong đợi
   - ✅ Validation passed: true

### Kiểm tra Chi tiết

#### **Backup Function Test Results**
Khi test thành công, bạn sẽ thấy:
```json
{
  "statusCode": 200,
  "body": {
    "message": "Enhanced backup hoàn thành thành công",
    "backup_results": [
      {
        "table": "app-users",
        "filename": "dynamodb-backups/app-users-backup-2025-01-28-XX-XX-XX.json",
        "item_count": 3,
        "status": "success",
        "checksum": "abc123...",
        "timestamp": "2025-01-28T10:30:00.000Z"
      }
    ],
    "validation_results": [
      {
        "table": "app-users",
        "checksum_valid": true,
        "item_count_match": true,
        "status": "valid"
      }
    ]
  }
}
```

#### **Validator Function Test Results**
```json
{
  "statusCode": 200,
  "body": {
    "validation_results": [
      {
        "filename": "dynamodb-backups/app-users-backup-2025-01-28-XX-XX-XX.json",
        "table": "app-users",
        "overall_status": true,
        "checks": {
          "file_readable": true,
          "structure_valid": true,
          "checksum_valid": true,
          "item_count_positive": true,
          "timestamp_recent": true
        }
      }
    ],
    "overall_success": true,
    "total_files_validated": 1,
    "successful_validations": 1
  }
}
```

#### **Recovery Function Test Results**
```json
{
  "statusCode": 200,
  "body": {
    "message": "Recovery hoàn thành thành công",
    "table_name": "app-users",
    "backup_file": "dynamodb-backups/app-users-backup-2025-01-28-XX-XX-XX.json",
    "items_restored": 3,
    "validation_passed": true
  }
}
```

### Troubleshooting

#### **Lỗi thường gặp:**

1. **Environment Variables không được cấu hình**
   - Kiểm tra BACKUP_BUCKET, SUCCESS_TOPIC_ARN, FAILURE_TOPIC_ARN
   - Đảm bảo ARNs đúng định dạng

2. **Permission Errors**
   - Xác minh IAM role có đủ quyền
   - Kiểm tra S3 bucket permissions

3. **Timeout Errors**
   - Tăng timeout cho function
   - Kiểm tra kích thước dữ liệu

4. **Checksum Validation Failures**
   - Kiểm tra tính toàn vẹn dữ liệu
   - Xác minh quá trình serialization JSON
