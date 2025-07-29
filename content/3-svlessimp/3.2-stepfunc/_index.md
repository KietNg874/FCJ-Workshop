---
title : "Build Step Functions"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

1. **Navigate to Step Functions Console**
   - Go to https://console.aws.amazon.com/states/
   - Click **"Create state machine"**

![img](/FCJ-Workshop/images/3.svlessimp/state1.png)

2. **Choose Authoring Method**
   ```
   Authoring method: Write your workflow in code
   Type: Standard
   ```

3. **State Machine Configuration**
   ```
   State machine name: enhanced-backup-orchestration
   ```

### Define State Machine

Use this simplified workflow definition:

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

**Important**: Replace `YOUR_ACCOUNT` with your actual AWS Account ID

![img](/FCJ-Workshop/images/3.svlessimp/state2.png)

### Configure IAM Role

1. **Use existing role**: `StepFunctionsBackupRole`
2. **Review permissions** to ensure it can invoke Lambda and publish to SNS

![img](/FCJ-Workshop/images/3.svlessimp/state3.png)
