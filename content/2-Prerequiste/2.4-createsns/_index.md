---
title : "Create SNS Notification"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 2.4 </b> "
---

### Objectives
- Create SNS topics for notifications
- Configure email subscriptions
- Test notification delivery

### Create SNS Topics

**Success Notifications Topic**
1. **Go to SNS Console** → **Topics** → **"Create topic"**
2. **Configuration**:
   ```
   Type: Standard
   Name: backup-success-notifications
   Display name: Backup Success Notifications
   ```
![img](/FCJ-Workshop/images/2.prerequisite/sns1.png)

3. **Access Policy**: Use default (only topic owner can publish)
4. **Encryption**: Leave default (no encryption needed for notifications)
5. Click **"Create topic"**

**Failure Notifications Topic**
   ```
   Type: Standard
   Name: backup-failure-notifications
   Display name: Backup Failure Notifications
   Same settings as success topic
   ```

### Create Email Subscriptions

**For each topic**:
1. **Click on topic name** → **Subscriptions** → **"Create subscription"**
2. **Configuration**:
   ```
   Protocol: Email
   Endpoint: your-email@example.com
   ```
3. Click **"Create subscription"**
4. **Check your email** and click the confirmation link
5. **Verify status** shows "Confirmed"

![img](/FCJ-Workshop/images/2.prerequisite/sns2.png)
