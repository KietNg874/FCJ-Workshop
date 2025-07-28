---
title : "Tạo SNS Notification"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 2.4 </b> "
---

### Mục tiêu
- Tạo SNS topics cho thông báo
- Cấu hình đăng ký email
- Kiểm thử việc gửi thông báo

### Tạo SNS Topics

**Success Notifications Topic**
1. **Truy cập SNS Console** → **Topics** → **"Create topic"**
2. **Cấu hình**:
   ```
   Type: Standard
   Name: backup-success-notifications
   Display name: Backup Success Notifications
   ```
![img](/images/2.prerequisite/sns1.png)

3. **Access Policy**: Sử dụng mặc định (chỉ chủ sở hữu topic có thể publish)
4. **Encryption**: Để mặc định (không cần mã hóa cho thông báo)
5. Nhấp **"Create topic"**

**Failure Notifications Topic**
   ```
   Type: Standard
   Name: backup-failure-notifications
   Display name: Backup Failure Notifications
   Cài đặt giống như success topic
   ```

### Tạo Email Subscriptions

**Cho mỗi topic**:
1. **Nhấp vào tên topic** → **Subscriptions** → **"Create subscription"**
2. **Cấu hình**:
   ```
   Protocol: Email
   Endpoint: your-email@example.com
   ```
3. Nhấp **"Create subscription"**
4. **Kiểm tra email của bạn** và nhấp vào liên kết xác nhận
5. **Xác minh trạng thái** hiển thị "Confirmed"

![img](/images/2.prerequisite/sns2.png)
