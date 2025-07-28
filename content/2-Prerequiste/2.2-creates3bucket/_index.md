---
title : "Create S3 Buckets with Cross-Region Replication"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

### Objectives
- Create secure S3 buckets with encryption
- Configure cross-region replication for disaster recovery
- Set up proper folder structure for organized backups

### Create Primary Backup Bucket

1. **Navigate to S3 Console**
   - Go to https://console.aws.amazon.com/s3/
   - Ensure you're in your primary region (e.g., us-east-1)
   - Click **"Create bucket"**

2. **Basic Configuration**
   ```
   Bucket name: serverless-backup-primary-[yourname]
   Region: us-east-1 (or your preferred primary region)
   ```

{{% notice warning %}}
**Warning**: Remember to replace **[yourname]** with your actual name or initials.
{{% /notice %}}

3. **Security Configuration**
   - **Block Public Access**: ✅ Keep all options checked
   - **Bucket Versioning**: ✅ Enable
   - **Tags**: Optional

4. **Default Encryption**
   ```
   Encryption type: SSE-S3 (Server-side encryption with Amazon S3-managed keys)
   Bucket Key: ✅ Enable (for cost optimization)
   ```

5. **Advanced Settings**
   - Object Lock: ❌ Disabled for this workshop
   - Click **"Create bucket"**

![Setup S3 Bucket](/images/2.prerequisite/s3b1.png)

### Create Secondary Region Bucket

1. **Switch to Secondary Region**
   - Change AWS Console region to us-west-2 (or your preferred secondary region)
   - Navigate to S3 Console

2. **Create Secondary Bucket**
   ```
   Bucket name: serverless-backup-secondary-[yourname]
   Region: us-west-2
   Same security settings as primary bucket
   ```

### Configure Cross-Region Replication

1. **Return to Primary Region** (us-east-1)
2. **Open Primary Bucket** → **Management** tab
3. **Create Replication Rule**
   ```
   Rule name: backup-replication
   Status: ✅ Enabled
   Priority: 1
   ```
![S3 Folders](/images/2.prerequisite/s3b3.png)
4. **Source Configuration**
   ```
   Source: ✅ Entire bucket
   Storage class: ✅ Replicate objects encrypted with SSE-S3
   ```

5. **Destination Configuration**
   ```
   Destination bucket: serverless-backup-secondary-[yourname]
   Destination storage class: Same as source
   ```

6. **IAM Role**
   - ✅ Create new role (let AWS create it automatically)
   - Role name will be: `replication-role-[timestamp]`

7. **Additional Options**
   ```
   Delete marker replication: ❌ Disabled
   Replica modification sync: ❌ Disabled
   ```

8. Click **"Save"**

### Create Folder Structure

In your **primary bucket**, create these folders:
1. Click **"Create folder"**
2. Create the following folders:
   - `dynamodb-backups/`
   - `lambda-code/`
   - `config-backups/`
   - `validation-reports/`

![S3 Folders](/images/2.prerequisite/s3b2.png)
