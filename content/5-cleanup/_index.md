---
title: "Cleanup"
weight: 50
chapter: false
pre: "<b>5. </b>"
---
### Complete Resource List

**Lambda Functions (3)**
- `enhanced-dynamodb-backup`
- `backup-validator`
- `enhanced-recovery-orchestrator`

**Step Functions (1)**
- `enhanced-backup-orchestration`

**EventBridge Rules (2)**
- `daily-backup-schedule`
- `weekly-full-backup`

**API Gateway (1)**
- `backup-recovery-api` (with dev stage)

**S3 Buckets (2)**
- `serverless-backup-primary-[yourname]`
- `serverless-backup-secondary-[yourname]`

**DynamoDB Tables (3)**
- `app-users`
- `app-orders`
- `backup-metadata`

**SNS Topics (2)**
- `backup-success-notifications`
- `backup-failure-notifications`

**IAM Roles (2+)**
- `EnhancedBackupLambdaRole`
- `StepFunctionsBackupRole`
- Auto-created replication role
- Auto-created EventBridge role

**CloudWatch Resources**
- Custom dashboard: `Enhanced-Backup-DR-Operations`
- CloudWatch alarms (2+)
- Log groups (auto-created for Lambda functions)

### Dependency Mapping

**Cleanup Order** (to avoid dependency errors):
1. EventBridge Rules (stop scheduled executions)
2. API Gateway (remove external access)
3. Step Functions (stop workflow orchestration)
4. Lambda Functions (remove compute resources)
5. CloudWatch Alarms and Dashboard
6. SNS Topics and Subscriptions
7. S3 Buckets (remove data and buckets)
8. DynamoDB Tables (remove data sources)
9. IAM Roles and Policies (remove permissions)

{{% notice tip %}}
**Cleanup Order**: Following the correct order prevents dependency errors. Always remove dependent resources before the resources they depend on.
{{% /notice %}}

### Stop Scheduled Operations

**Disable EventBridge Rules**
1. **Navigate to EventBridge Console**
2. **For each rule** (`daily-backup-schedule`, `weekly-full-backup`):
   - Click on rule name
   - Click **"Disable"**
   - Confirm disabling
   - Click **"Delete"**
   - Confirm deletion


### Remove API Gateway

1. **Navigate to API Gateway Console**
2. **Select** `backup-recovery-api`
3. **Actions** → **Delete API**
4. **Type API name** to confirm
5. **Click "Delete"**


### Delete Step Functions

1. **Navigate to Step Functions Console**
2. **Select** `enhanced-backup-orchestration`
3. **Delete** → **Confirm deletion**


### Delete Lambda Functions

**For each Lambda function**:
1. **Navigate to Lambda Console**
2. **Select function** (`enhanced-dynamodb-backup`, `backup-validator`, `enhanced-recovery-orchestrator`)
3. **Actions** → **Delete function**
4. **Type "delete"** to confirm
5. **Click "Delete"**

{{% notice warning %}}
**Lambda Deletion**: When you delete a Lambda function, all versions and aliases are also deleted. Make sure you don't need any of the code before deletion.
{{% /notice %}}

### Remove CloudWatch Resources

**Delete Dashboard**
1. **Navigate to CloudWatch Console** → **Dashboards**
2. **Select** `Enhanced-Backup-DR-Operations`
3. **Delete** → **Confirm**

**Delete Alarms**
1. **Go to** **Alarms** section
2. **Select all workshop-related alarms**
3. **Actions** → **Delete**
4. **Confirm deletion**

**Delete Log Groups** (optional - they'll expire automatically)
1. **Go to** **Log groups**
2. **Find Lambda log groups**: `/aws/lambda/enhanced-*`
3. **Select and delete** if desired

### Delete SNS Topics

**For each SNS topic**:
1. **Navigate to SNS Console** → **Topics**
2. **Select topic** (`backup-success-notifications`, `backup-failure-notifications`)
3. **Delete** → **Confirm deletion**

**Note**: Email subscriptions will be automatically removed

### Empty and Delete S3 Buckets

{{% notice warning %}}
**Important**: S3 buckets must be empty before deletion. This includes all object versions if versioning is enabled.
{{% /notice %}}

**For Primary Bucket**:
1. **Navigate to S3 Console**
2. **Open** `serverless-backup-primary-[yourname]`
3. **Select all objects** → **Delete**
4. **Type "permanently delete"** → **Confirm**
5. **Go back to bucket list**
6. **Select bucket** → **Delete**
7. **Type bucket name** → **Confirm deletion**

**For Secondary Bucket**:
1. **Switch to us-west-2 region**
2. **Repeat same process** for `serverless-backup-secondary-[yourname]`

### Delete DynamoDB Tables

**For each table**:
1. **Navigate to DynamoDB Console**
2. **Select table** (`app-users`, `app-orders`, `backup-metadata`)
3. **Delete** → **Confirm deletion**
4. **Type "confirm"** → **Delete table**

### Delete IAM Roles and Policies

**Delete Custom Policies**:
1. **Navigate to IAM Console** → **Policies**
2. **Find custom policies**:
   - `EnhancedBackupPolicy`
   - Any Step Functions policies created
3. **Select policy** → **Delete** → **Confirm**

**Delete IAM Roles**:
1. **Go to** **Roles**
2. **Find workshop roles**:
   - `EnhancedBackupLambdaRole`
   - `StepFunctionsBackupRole`
   - Any auto-created replication roles
3. **Select role** → **Delete** → **Confirm**

{{% notice info %}}
**IAM Cleanup**: AWS automatically deletes service-linked roles when they're no longer needed. You only need to delete the roles you explicitly created.
{{% /notice %}}


