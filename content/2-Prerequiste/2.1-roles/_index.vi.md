---
title : "Tạo IAM Roles"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---


### Tạo Enhanced Lambda Execution Role

1. **Điều hướng đến IAM Console**
   - Truy cập https://console.aws.amazon.com/iam/
   - Nhấp **Roles** → **Create role**

2. **Cấu hình Trust Relationship**
   ```
   Trusted entity type: AWS service
   Service: Lambda
   ```

3. **Cấu hình Role**
   ```
   Role name: EnhancedBackupLambdaRole
   Description: Role for enhanced backup Lambda functions
   ```

### Gắn Basic Lambda Policy

1. **Tìm kiếm và gắn**: `AWSLambdaBasicExecutionRole`
   - Điều này cung cấp quyền CloudWatch Logs cơ bản
   - Cho phép các hàm Lambda ghi logs

![S3 Folders](/FCJ-Workshop/images/2.prerequisite/IAM1.png)

### Tạo Custom Backup Policy

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

4. **Gắn Policy** vào `EnhancedBackupLambdaRole`

### Tạo Step Functions Role

1. **Tạo role khác**:
   ```
   Trusted entity: AWS service → Step Functions
   Role name: StepFunctionsBackupRole
   Description: Role for Step Functions backup orchestration
   ```

2. **Tạo Step Functions Policy**:
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
