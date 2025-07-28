---
title: "Dọn dẹp"
weight: 50
chapter: false
pre: "<b>5. </b>"
---
### Danh sách Tài nguyên Hoàn chỉnh

**Lambda Functions (3)**
- `enhanced-dynamodb-backup`
- `backup-validator`
- `enhanced-recovery-orchestrator`

**Step Functions (1)**
- `enhanced-backup-orchestration`

**EventBridge Rules (2)**
- `daily-backup-schedule`
- `weekly-full-backup`

**API Gateway (1)**
- `backup-recovery-api` (với dev stage)

**S3 Buckets (2)**
- `serverless-backup-primary-[yourname]`
- `serverless-backup-secondary-[yourname]`

**DynamoDB Tables (3)**
- `app-users`
- `app-orders`
- `backup-metadata`

**SNS Topics (2)**
- `backup-success-notifications`
- `backup-failure-notifications`

**IAM Roles (2+)**
- `EnhancedBackupLambdaRole`
- `StepFunctionsBackupRole`
- Role sao chép được tạo tự động
- Role EventBridge được tạo tự động

**CloudWatch Resources**
- Dashboard tùy chỉnh: `Enhanced-Backup-DR-Operations`
- CloudWatch alarms (2+)
- Log groups (được tạo tự động cho các hàm Lambda)

### Ánh xạ Phụ thuộc

**Thứ tự Dọn dẹp** (để tránh lỗi phụ thuộc):
1. EventBridge Rules (dừng thực thi theo lịch)
2. API Gateway (loại bỏ truy cập bên ngoài)
3. Step Functions (dừng điều phối quy trình)
4. Lambda Functions (loại bỏ tài nguyên tính toán)
5. CloudWatch Alarms và Dashboard
6. SNS Topics và Subscriptions
7. S3 Buckets (loại bỏ dữ liệu và buckets)
8. DynamoDB Tables (loại bỏ nguồn dữ liệu)
9. IAM Roles và Policies (loại bỏ quyền)

{{% notice tip %}}
**Thứ tự Dọn dẹp**: Tuân theo thứ tự đúng ngăn ngừa lỗi phụ thuộc. Luôn loại bỏ tài nguyên phụ thuộc trước các tài nguyên mà chúng phụ thuộc vào.
{{% /notice %}}

### Dừng Hoạt động theo Lịch

**Vô hiệu hóa EventBridge Rules**
1. **Điều hướng đến EventBridge Console**
2. **Cho mỗi rule** (`daily-backup-schedule`, `weekly-full-backup`):
   - Nhấp vào tên rule
   - Nhấp **"Disable"**
   - Xác nhận vô hiệu hóa
   - Nhấp **"Delete"**
   - Xác nhận xóa


### Loại bỏ API Gateway

1. **Điều hướng đến API Gateway Console**
2. **Chọn** `backup-recovery-api`
3. **Actions** → **Delete API**
4. **Nhập tên API** để xác nhận
5. **Nhấp "Delete"**


### Xóa Step Functions

1. **Điều hướng đến Step Functions Console**
2. **Chọn** `enhanced-backup-orchestration`
3. **Delete** → **Xác nhận xóa**


### Xóa Lambda Functions

**Cho mỗi Lambda function**:
1. **Điều hướng đến Lambda Console**
2. **Chọn function** (`enhanced-dynamodb-backup`, `backup-validator`, `enhanced-recovery-orchestrator`)
3. **Actions** → **Delete function**
4. **Nhập "delete"** để xác nhận
5. **Nhấp "Delete"**

{{% notice warning %}}
**Xóa Lambda**: Khi bạn xóa một hàm Lambda, tất cả versions và aliases cũng bị xóa. Đảm bảo bạn không cần bất kỳ code nào trước khi xóa.
{{% /notice %}}

### Loại bỏ CloudWatch Resources

**Xóa Dashboard**
1. **Điều hướng đến CloudWatch Console** → **Dashboards**
2. **Chọn** `Enhanced-Backup-DR-Operations`
3. **Delete** → **Xác nhận**

**Xóa Alarms**
1. **Truy cập** **Alarms** section
2. **Chọn tất cả alarms liên quan đến workshop**
3. **Actions** → **Delete**
4. **Xác nhận xóa**

**Xóa Log Groups** (tùy chọn - chúng sẽ tự động hết hạn)
1. **Truy cập** **Log groups**
2. **Tìm Lambda log groups**: `/aws/lambda/enhanced-*`
3. **Chọn và xóa** nếu muốn

### Xóa SNS Topics

**Cho mỗi SNS topic**:
1. **Điều hướng đến SNS Console** → **Topics**
2. **Chọn topic** (`backup-success-notifications`, `backup-failure-notifications`)
3. **Delete** → **Xác nhận xóa**

**Lưu ý**: Email subscriptions sẽ được tự động loại bỏ

### Làm trống và Xóa S3 Buckets

{{% notice warning %}}
**Quan trọng**: S3 buckets phải trống trước khi xóa. Điều này bao gồm tất cả object versions nếu versioning được bật.
{{% /notice %}}

**Cho Primary Bucket**:
1. **Điều hướng đến S3 Console**
2. **Mở** `serverless-backup-primary-[yourname]`
3. **Chọn tất cả objects** → **Delete**
4. **Nhập "permanently delete"** → **Xác nhận**
5. **Quay lại danh sách bucket**
6. **Chọn bucket** → **Delete**
7. **Nhập tên bucket** → **Xác nhận xóa**

**Cho Secondary Bucket**:
1. **Chuyển đến vùng us-west-2**
2. **Lặp lại quy trình tương tự** cho `serverless-backup-secondary-[yourname]`

### Xóa DynamoDB Tables

**Cho mỗi table**:
1. **Điều hướng đến DynamoDB Console**
2. **Chọn table** (`app-users`, `app-orders`, `backup-metadata`)
3. **Delete** → **Xác nhận xóa**
4. **Nhập "confirm"** → **Delete table**

### Xóa IAM Roles và Policies

**Xóa Custom Policies**:
1. **Điều hướng đến IAM Console** → **Policies**
2. **Tìm custom policies**:
   - `EnhancedBackupPolicy`
   - Bất kỳ Step Functions policies nào được tạo
3. **Chọn policy** → **Delete** → **Xác nhận**

**Xóa IAM Roles**:
1. **Truy cập** **Roles**
2. **Tìm workshop roles**:
   - `EnhancedBackupLambdaRole`
   - `StepFunctionsBackupRole`
   - Bất kỳ auto-created replication roles nào
3. **Chọn role** → **Delete** → **Xác nhận**

{{% notice info %}}
**Dọn dẹp IAM**: AWS tự động xóa service-linked roles khi chúng không còn cần thiết. Bạn chỉ cần xóa các roles mà bạn đã tạo một cách rõ ràng.
{{% /notice %}}
