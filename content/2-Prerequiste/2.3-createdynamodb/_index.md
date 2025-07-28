---
title : "Create DynamoDB"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---

### Objectives
- Create DynamoDB tables with security best practices
- Enable point-in-time recovery and streams
- Add sample data for testing backup functionality

### Create Application Tables

**Table 1: app-orders**
1. **Go to DynamoDB Console** → **"Create table"**
2. **Basic Configuration**
   ```
   Table name: app-orders
   Partition key: orderId (String)
   ```
![Img](/images/2.prerequisite/dynamodb1.png)
3. **Table Settings**
   ```
   Table class: DynamoDB Standard
   Billing mode: On-demand
   ```
4. **Advanced Settings**
   ```
   Encryption at rest: ✅ Enabled (AWS owned key)
   Point-in-time recovery: ✅ Enable
   DynamoDB Streams: ✅ Enable (New and old images)
   ```
5. Click **"Create table"**

**Table 2: app-users**
1. **Repeat the process** with:
   ```
   Table name: app-users
   Partition key: userId (String)
   Same advanced settings as above
   ```

**Table 3: backup-metadata**
1. **Create with**:
   ```
   Table name: backup-metadata
   Partition key: backupId (String)
   Sort key: timestamp (String)
   Same advanced settings as above
   ```

### Add Sample Data

**For app-users table:**
1. Go to table → **Items** tab → **"Create item"**
2. Add these sample items:

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

**For app-orders table:**
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

**For backup-metadata table:**
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
**Data Consistency**: Ensure all sample data follows the same format and structure. This will be important for testing backup and recovery operations.
{{% /notice %}}
