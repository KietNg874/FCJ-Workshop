---
title : "Tạo DynamoDB"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---

### Mục tiêu
- Tạo bảng DynamoDB với thực hành bảo mật tốt nhất
- Bật point-in-time recovery và streams
- Thêm dữ liệu mẫu để kiểm thử chức năng sao lưu

### Tạo Application Tables

**Bảng 1: app-orders**
1. **Truy cập DynamoDB Console** → **"Create table"**
2. **Cấu hình Cơ bản**
   ```
   Table name: app-orders
   Partition key: orderId (String)
   ```
![Img](/images/2.prerequisite/dynamodb1.png)
3. **Cài đặt Bảng**
   ```
   Table class: DynamoDB Standard
   Billing mode: On-demand
   ```
4. **Cài đặt Nâng cao**
   ```
   Encryption at rest: ✅ Bật (AWS owned key)
   Point-in-time recovery: ✅ Bật
   DynamoDB Streams: ✅ Bật (New and old images)
   ```
5. Nhấp **"Create table"**

**Bảng 2: app-users**
1. **Lặp lại quy trình** với:
   ```
   Table name: app-users
   Partition key: userId (String)
   Cài đặt nâng cao giống như trên
   ```

**Bảng 3: backup-metadata**
1. **Tạo với**:
   ```
   Table name: backup-metadata
   Partition key: backupId (String)
   Sort key: timestamp (String)
   Cài đặt nâng cao giống như trên
   ```

### Thêm Dữ liệu Mẫu

**Cho bảng app-users:**
1. Truy cập bảng → **Items** tab → **"Create item"**
2. Thêm các item mẫu này:

```json
{
  "userId": "user001",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "createdAt": "2025-01-01T00:00:00Z",
  "status": "active"
}
```

```json
{
  "userId": "user002", 
  "name": "Jane Smith",
  "email": "jane.smith@example.com",
  "createdAt": "2025-01-02T00:00:00Z",
  "status": "active"
}
```

```json
{
  "userId": "user003",
  "name": "Bob Johnson", 
  "email": "bob.johnson@example.com",
  "createdAt": "2025-01-03T00:00:00Z",
  "status": "inactive"
}
```

**Cho bảng app-orders:**
```json
{
  "orderId": "order001",
  "userId": "user001",
  "amount": 99.99,
  "status": "completed",
  "createdAt": "2025-01-01T12:00:00Z"
}
```

```json
{
  "orderId": "order002",
  "userId": "user002", 
  "amount": 149.50,
  "status": "pending",
  "createdAt": "2025-01-02T14:30:00Z"
}
```

```json
{
  "orderId": "order003",
  "userId": "user001",
  "amount": 75.25,
  "status": "completed", 
  "createdAt": "2025-01-03T09:15:00Z"
}
```

**Cho bảng backup-metadata:**
```json
{
  "backupId": "backup001",
  "timestamp": "2025-01-01T00:00:00Z",
  "status": "completed",
  "tableCount": 3,
  "totalItems": 6
}
```

{{% notice warning %}}
**Tính nhất quán Dữ liệu**: Đảm bảo tất cả dữ liệu mẫu tuân theo cùng định dạng và cấu trúc. Điều này sẽ quan trọng cho việc kiểm thử các hoạt động sao lưu và khôi phục.
{{% /notice %}}
