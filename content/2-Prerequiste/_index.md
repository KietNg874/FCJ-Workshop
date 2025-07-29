---
title : "Prerequisites"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

Before building our serverless backup solution, we need to establish the foundational AWS services and security configurations. This section will guide you through setting up the essential infrastructure components that our backup system will depend on.

![Prerequisites Setup](/FCJ-Workshop/images/2.prerequisite/001-setup.png)

In this prerequisites section, you'll establish the core infrastructure for your serverless backup solution:

### Security Foundation
- **IAM Roles**: Service roles with least-privilege permissions for Lambda functions, S3 access, and DynamoDB operations
- **Cross-service permissions**: Secure access patterns between AWS services

### Data Infrastructure  
- **DynamoDB Tables**: Three tables to simulate real application data
  - App Users Table (user profiles and preferences)
  - App Orders Table (transaction and order data)
  - Backup Metadata Table (tracking backup operations)

### Storage Infrastructure
- **Primary S3 Bucket**: Main backup storage in us-east-1 region
- **Secondary S3 Bucket**: Disaster recovery storage in us-west-2 region
- **Cross-region replication**: Automated data replication for disaster recovery

### Notification System
- **SNS Topic**: Alert system for backup success/failure notifications
- **Email subscriptions**: Real-time notifications for operational events

### Content
2.1. [Create IAM Roles](2.1-roles/) \
2.2. [Create S3 Buckets with Cross-Region Replication](2.2-creates3bucket/) \
2.3. [Create DynamoDB](2.3-createdynamodb/) \
2.4. [Create SNS Notification](2.4-createsns/) 
