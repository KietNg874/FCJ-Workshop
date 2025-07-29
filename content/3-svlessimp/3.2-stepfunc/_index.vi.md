---
title : "Xây dựng Step Functions"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

### Tạo Step Functions State Machine

1. **Điều hướng đến Step Functions Console**
   - Truy cập https://console.aws.amazon.com/states/
   - Nhấp **"Create state machine"**

![img](/FCJ-Workshop/images/3.svlessimp/state1.png)

2. **Chọn Authoring Method**
   ```
   Authoring method: Write your workflow in code
   Type: Standard
   ```

3. **Cấu hình State Machine**
   ```
   State machine name: enhanced-backup-orchestration
   ```

### Định nghĩa State Machine

Sử dụng định nghĩa quy trình đơn giản này:

```json
{
  "Comment": "Enhanced Backup Orchestration - Simplified",
  "StartAt": "BackupTables",
  "States": {
    "BackupTables": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:YOUR_ACCOUNT:function:enhanced-dynamodb-backup",
      "Parameters": {
        "tables": ["app-users", "app-orders", "backup-metadata"]
      },
      "Retry": [
        {
          "ErrorEquals": ["States.TaskFailed"],
          "IntervalSeconds": 2,
          "MaxAttempts": 3,
          "BackoffRate": 2.0
        }
      ],
      "Catch": [
        {
          "ErrorEquals": ["States.ALL"],
          "Next": "BackupFailureHandler",
          "ResultPath": "$.error"
        }
      ],
      "Next": "CheckBackupSuccess"
    },
    "CheckBackupSuccess": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.statusCode",
          "NumericEquals": 200,
          "Next": "SendSuccessNotification"
        }
      ],
      "Default": "BackupFailureHandler"
    },
    "SendSuccessNotification": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "arn:aws:sns:us-east-1:YOUR_ACCOUNT:backup-success-notifications",
        "Message.$": "$.body",
        "Subject": "Backup Workflow Success"
      },
      "End": true
    },
    "BackupFailureHandler": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "arn:aws:sns:us-east-1:YOUR_ACCOUNT:backup-failure-notifications",
        "Message.$": "$.error",
        "Subject": "Backup Workflow Failure"
      },
      "End": true
    }
  }
}
```

**Quan trọng**: Thay thế `YOUR_ACCOUNT` bằng AWS Account ID thực tế của bạn

![img](/FCJ-Workshop/images/3.svlessimp/state2.png)

### Cấu hình IAM Role

1. **Sử dụng role hiện có**: `StepFunctionsBackupRole`
2. **Xem lại quyền** để đảm bảo nó có thể gọi Lambda và publish đến SNS

![img](/FCJ-Workshop/images/3.svlessimp/state3.png)
