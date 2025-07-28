---
title : "Build Event Bridge Scheduling"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3.3. </b> "
---

### Objectives
- Create automated backup schedules
- Configure EventBridge rules
- Set up different backup frequencies

### Create Daily Backup Schedule

1. **Navigate to EventBridge Console**
   - Go to https://console.aws.amazon.com/events/
   - Click **Rules** → **"Create rule"**

2. **Rule Configuration**
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
   *This runs at 2 AM UTC every day*

![img](/images/3.svlessimp/bridge1.png)

4. **Target Configuration**
   ```
   Target type: AWS service
   Service: Step Functions state machine
   State machine: enhanced-backup-orchestration
   ```

5. **Input Configuration**
   ```
   Configure input: Constant (JSON text)
   ```
   ```json
   {
     "tables": ["app-users", "app-orders", "backup-metadata"]
   }
   ```

6. **Create IAM role** for EventBridge to invoke Step Functions
