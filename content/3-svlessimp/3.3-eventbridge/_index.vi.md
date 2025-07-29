---
title : "Xây dựng Event Bridge Scheduling"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3.3. </b> "
---

### Tạo Daily Backup Schedule

1. **Điều hướng đến EventBridge Console**
   - Truy cập https://console.aws.amazon.com/events/
   - Nhấp **Rules** → **"Create rule"**

2. **Cấu hình Rule**
   ```
   Name: daily-backup-schedule
   Description: Trigger daily backup at 2 AM UTC
   Event bus: default
   Rule type: Schedule
   ```

3. **Schedule Pattern**
   ```
   Schedule expression: cron(0 2 * * ? *)
   ```
   *Điều này chạy lúc 2 giờ sáng UTC mỗi ngày*

![img](/FCJ-Workshop/images/3.svlessimp/bridge1.png)

4. **Cấu hình Target**
   ```
   Target type: AWS service
   Service: Step Functions state machine
   State machine: enhanced-backup-orchestration
   ```

5. **Cấu hình Input**
   ```
   Configure input: Constant (JSON text)
   ```
   ```json
   {
     "tables": ["app-users", "app-orders", "backup-metadata"]
   }
   ```

6. **Tạo IAM role** cho EventBridge để gọi Step Functions
