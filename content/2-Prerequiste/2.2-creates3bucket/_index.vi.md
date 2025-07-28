---
title : "Tạo S3 Buckets với Cross-Region Replication"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

### Mục tiêu
- Tạo bucket S3 an toàn với mã hóa
- Cấu hình sao chép đa vùng cho khôi phục thảm họa
- Thiết lập cấu trúc thư mục phù hợp cho sao lưu có tổ chức

### Tạo Primary Backup Bucket

1. **Điều hướng đến S3 Console**
   - Truy cập https://console.aws.amazon.com/s3/
   - Đảm bảo bạn đang ở vùng chính (ví dụ: us-east-1)
   - Nhấp **"Create bucket"**

2. **Cấu hình Cơ bản**
   ```
   Bucket name: serverless-backup-primary-[yourname]
   Region: us-east-1 (hoặc vùng chính ưa thích của bạn)
   ```

{{% notice warning %}}
**Cảnh báo**: Nhớ thay thế **[yourname]** bằng tên thật hoặc chữ cái đầu của bạn.
{{% /notice %}}

3. **Cấu hình Bảo mật**
   - **Block Public Access**: ✅ Giữ tất cả tùy chọn được đánh dấu
   - **Bucket Versioning**: ✅ Bật
   - **Tags**: Tùy chọn

4. **Default Encryption**
   ```
   Encryption type: SSE-S3 (Server-side encryption với Amazon S3-managed keys)
   Bucket Key: ✅ Bật (để tối ưu hóa chi phí)
   ```

5. **Cài đặt Nâng cao**
   - Object Lock: ❌ Tắt cho workshop này
   - Nhấp **"Create bucket"**

![Setup S3 Bucket](/FCJ-Workshop/images/2.prerequisite/s3b1.png)

### Tạo Secondary Region Bucket

1. **Chuyển đến Secondary Region**
   - Thay đổi vùng AWS Console thành us-west-2 (hoặc vùng phụ ưa thích của bạn)
   - Điều hướng đến S3 Console

2. **Tạo Secondary Bucket**
   ```
   Bucket name: serverless-backup-secondary-[yourname]
   Region: us-west-2
   Cài đặt bảo mật giống như primary bucket
   ```

### Cấu hình Cross-Region Replication

1. **Quay lại Primary Region** (us-east-1)
2. **Mở Primary Bucket** → **Management** tab
3. **Tạo Replication Rule**
   ```
   Rule name: backup-replication
   Status: ✅ Bật
   Priority: 1
   ```
![S3 Folders](/FCJ-Workshop/images/2.prerequisite/s3b3.png)

4. **Cấu hình Source**
   ```
   Source: ✅ Toàn bộ bucket
   Storage class: ✅ Sao chép objects được mã hóa với SSE-S3
   ```

5. **Cấu hình Destination**
   ```
   Destination bucket: serverless-backup-secondary-[yourname]
   Destination storage class: Giống như source
   ```

6. **IAM Role**
   - ✅ Tạo role mới (để AWS tự động tạo)
   - Tên role sẽ là: `replication-role-[timestamp]`

7. **Tùy chọn Bổ sung**
   ```
   Delete marker replication: ❌ Tắt
   Replica modification sync: ❌ Tắt
   ```

8. Nhấp **"Save"**

### Tạo Cấu trúc Thư mục

Trong **primary bucket** của bạn, tạo các thư mục này:
1. Nhấp **"Create folder"**
2. Tạo các thư mục sau:
   - `dynamodb-backups/`
   - `lambda-code/`
   - `config-backups/`
   - `validation-reports/`

![S3 Folders](/FCJ-Workshop/images/2.prerequisite/s3b2.png)
