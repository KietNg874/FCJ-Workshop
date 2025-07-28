---
title : "Xây dựng API Gateway"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 3.4. </b> "
---

### Mục tiêu
- Tạo REST API cho các hoạt động sao lưu
- Cấu hình bảo mật và tích hợp API
- Kiểm thử API endpoints

### Tạo REST API

1. **Điều hướng đến API Gateway Console**
   - Truy cập https://console.aws.amazon.com/apigateway/
   - Nhấp **"Create API"**

2. **Chọn API Type**
   ```
   API type: REST API (không phải REST API Private)
   ```

3. **Cấu hình API**
   ```
   API name: backup-recovery-api
   Description: API for backup and recovery operations
   Endpoint type: Regional
   ```

![img](/images/3.svlessimp/api1.png)

### Tạo API Resources và Methods

**Resource 1: /backup**
1. **Actions** → **Create Resource**
   ```
   Resource name: backup
   Resource path: /backup
   Enable API Gateway CORS: ✅ (tùy chọn)
   ```

2. **Actions** → **Create Method** → **POST**
3. **Cấu hình Integration**:
   ```
   Integration type: Lambda Function
   Use Lambda Proxy integration: ✅ Đánh dấu
   Lambda Region: us-east-1
   Lambda Function: enhanced-dynamodb-backup
   ```

![img](/images/3.svlessimp/api2.png)

**Resource 2: /recovery**
1. **Tạo resource**: `/recovery`
2. **Tạo method**: **POST**
3. **Integration**: `enhanced-recovery-orchestrator`

**Resource 3: /status**
1. **Tạo resource**: `/status`
2. **Tạo method**: **GET**
3. **Integration**: Tạo một status function đơn giản hoặc sử dụng mock integration

![img](/images/3.svlessimp/api3.png)

### Deploy API

1. **Actions** → **Deploy API**
2. **Cấu hình Deployment**:
   ```
   Deployment stage: dev
   Stage description: Development stage for backup API
   ```

![img](/images/3.svlessimp/api4.png)
