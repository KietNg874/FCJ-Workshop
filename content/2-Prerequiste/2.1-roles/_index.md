---
title : "Create IAM Roles"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---


### Create Enhanced Lambda Execution Role

1. **Navigate to IAM Console**
   - Go to https://console.aws.amazon.com/iam/
   - Click **Roles** → **Create role**

2. **Configure Trust Relationship**
   ```
   Trusted entity type: AWS service
   Service: Lambda
   ```

3. **Role Configuration**
   ```
   Role name: EnhancedBackupLambdaRole
   Description: Role for enhanced backup Lambda functions
   ```

### Attach Basic Lambda Policy

1. **Search and attach**: `AWSLambdaBasicExecutionRole`
   - This provides basic CloudWatch Logs permissions
   - Allows Lambda functions to write logs

![S3 Folders](/FCJ-Workshop/images/2.prerequisite/IAM1.png)

### Create Custom Backup Policy

1. **Create Policy** → **JSON** tab
2. **Policy Name**: `EnhancedBackupPolicy`
3. **Policy Document**:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:Scan",
                "dynamodb:Query",
                "dynamodb:DescribeTable",
                "dynamodb:DescribeStream",
                "dynamodb:GetRecords",
                "dynamodb:GetShardIterator",
                "dynamodb:ListStreams",
                "dynamodb:BatchWriteItem",
                "dynamodb:PutItem",
                "dynamodb:GetItem"
            ],
            "Resource": "arn:aws:dynamodb:*:*:table/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject",
                "s3:ListBucket",
                "s3:GetObjectVersion"
            ],
            "Resource": [
                "arn:aws:s3:::serverless-backup-*",
                "arn:aws:s3:::serverless-backup-*/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "sns:Publish"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:PutMetricData"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "xray:PutTraceSegments",
                "xray:PutTelemetryRecords"
            ],
            "Resource": "*"
        }
    ]
}
```

4. **Attach Policy** to `EnhancedBackupLambdaRole`

### Create Step Functions Role

1. **Create another role**:
   ```
   Trusted entity: AWS service → Step Functions
   Role name: StepFunctionsBackupRole
   Description: Role for Step Functions backup orchestration
   ```

2. **Create Step Functions Policy**:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "lambda:InvokeFunction"
            ],
            "Resource": "arn:aws:lambda:*:*:function:enhanced-*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "sns:Publish"
            ],
            "Resource": "*"
        }
    ]
}
```
