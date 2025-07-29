---
title : "Step Function Test"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 4.2. </b> "
---

### Test Step Functions Workflow

1. **Điều hướng đến Step Functions Console**
2. **Chọn** `enhanced-backup-orchestration` **state machine**
3. **Start execution**:
   ```
   Execution name: integration-test-1
   Input:
   ```
   ```json
   {
     "tables": ["app-users", "app-orders", "backup-metadata"]
   }
   ```

![img](/FCJ-Workshop/images/4.testing/TEST_stepexec.png)

4. **Giám sát execution**:
   - ✅ Theo dõi tiến trình workflow trực quan
   - ✅ Xác minh mỗi bước hoàn thành thành công
   - ✅ Kiểm tra thời gian execution < 2 phút
   - ✅ Xác minh nhận được thông báo thành công
   
![img](/FCJ-Workshop/images/4.testing/TEST_stepok.png)

### Test Workflow Error Handling

1. **Tạm thời sửa đổi** backup function để gây lỗi:
   - Thay đổi environment variable thành S3 bucket không hợp lệ
   - Hoặc sửa đổi code để throw exception

2. **Start execution** và xác minh:
   - ✅ Workflow bắt được lỗi
   - ✅ Failure handler thực thi
   - ✅ Thông báo thất bại được gửi
   - ✅ Execution hiển thị trạng thái failed với chi tiết

3. **Khôi phục** function về trạng thái hoạt động

### Test EventBridge Scheduling

1. **Tạm thời sửa đổi** daily backup rule:
   - Thay đổi schedule thành `cron(*/2 * * * ? *)` (mỗi 2 phút)
   - Lưu rule

2. **Đợi 2-3 phút** và xác minh:
   - ✅ Step Functions execution được kích hoạt tự động
   - ✅ Backup hoàn thành thành công
   - ✅ Backup files mới trong S3

3. **Reset schedule** về ban đầu `cron(0 2 * * ? *)`

### Test API Gateway Integration

1. **Điều hướng đến API Gateway Console**
2. **Test /backup endpoint**:
   - Truy cập `/backup` → `POST` → `TEST`
   - Request body:
   ```json
   {
     "tables": ["app-users"]
   }
   ```
   - ✅ Xác minh response 200
   - ✅ Kiểm tra response body chứa backup results

![img](/FCJ-Workshop/images/4.testing/TEST_api.png)

2. **Test /recovery endpoint**:
   - Truy cập `/recovery` → `POST` → `TEST`
   - Request body:
   ```json
   {
     "table_name": "app-users"
   }
   ```
   - ✅ Xác minh response recovery thành công

![img](/FCJ-Workshop/images/4.testing/TEST_apisucc.png)

{{% notice info %}}
**Integration Testing**: Điều này xác thực rằng tất cả các thành phần của bạn hoạt động cùng nhau một cách chính xác. Chú ý đến luồng dữ liệu giữa các dịch vụ và sự lan truyền lỗi.
{{% /notice %}}

### Kiểm tra Chi tiết

#### **Step Functions Execution Success**
Khi workflow thành công, bạn sẽ thấy:

**Visual Workflow:**
- 🟢 BackupTables: SUCCEEDED
- 🟢 CheckBackupSuccess: SUCCEEDED  
- 🟢 SendSuccessNotification: SUCCEEDED

**Execution Output:**
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
        "status": "success"
      },
      {
        "table": "app-orders", 
        "filename": "dynamodb-backups/app-orders-backup-2025-01-28-XX-XX-XX.json",
        "item_count": 3,
        "status": "success"
      },
      {
        "table": "backup-metadata",
        "filename": "dynamodb-backups/backup-metadata-backup-2025-01-28-XX-XX-XX.json", 
        "item_count": 1,
        "status": "success"
      }
    ]
  }
}
```

#### **Error Handling Test Results**
Khi có lỗi, workflow sẽ:

**Visual Workflow:**
- 🔴 BackupTables: FAILED
- 🟢 BackupFailureHandler: SUCCEEDED

**Error Details:**
```json
{
  "Error": "States.TaskFailed",
  "Cause": "Lambda function failed with error: S3 bucket not found"
}
```

#### **EventBridge Scheduled Execution**
- Execution được tự động tạo với tên như: `scheduled-execution-2025-01-28-XX-XX-XX`
- Input tự động từ EventBridge rule
- Thời gian execution khớp với schedule

#### **API Gateway Test Results**

**Backup Endpoint Success:**
```json
{
  "statusCode": 200,
  "body": {
    "message": "Enhanced backup hoàn thành thành công",
    "backup_results": [...],
    "validation_results": [...]
  }
}
```

**Recovery Endpoint Success:**
```json
{
  "statusCode": 200, 
  "body": {
    "message": "Recovery hoàn thành thành công",
    "table_name": "app-users",
    "items_restored": 3,
    "validation_passed": true
  }
}
```

### Troubleshooting Integration Issues

#### **Step Functions Failures:**
1. **Lambda Function Errors**: Kiểm tra CloudWatch logs của Lambda functions
2. **IAM Permission Issues**: Xác minh Step Functions role có quyền invoke Lambda
3. **Input/Output Mapping**: Kiểm tra data flow giữa các states

#### **EventBridge Issues:**
1. **Schedule Expression**: Xác minh cron expression đúng định dạng
2. **Target Configuration**: Kiểm tra Step Functions ARN trong rule target
3. **IAM Permissions**: EventBridge cần quyền để start Step Functions execution

#### **API Gateway Issues:**
1. **Lambda Integration**: Xác minh Lambda proxy integration được cấu hình đúng
2. **CORS Issues**: Bật CORS nếu cần thiết cho web clients
3. **Request/Response Mapping**: Kiểm tra data transformation
