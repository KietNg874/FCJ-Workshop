---
title : "Build API Gateway"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 3.4. </b> "
---

### Objectives
- Create REST API for backup operations
- Configure API security and integration
- Test API endpoints

### Create REST API

1. **Navigate to API Gateway Console**
   - Go to https://console.aws.amazon.com/apigateway/
   - Click **"Create API"**

2. **Choose API Type**
   ```
   API type: REST API (not REST API Private)
   ```

3. **API Configuration**
   ```
   API name: backup-recovery-api
   Description: API for backup and recovery operations
   Endpoint type: Regional
   ```

![img](/images/3.svlessimp/api1.png)

### Create API Resources and Methods

**Resource 1: /backup**
1. **Actions** → **Create Resource**
   ```
   Resource name: backup
   Resource path: /backup
   Enable API Gateway CORS: ✅ (optional)
   ```

2. **Actions** → **Create Method** → **POST**
3. **Integration Configuration**:
   ```
   Integration type: Lambda Function
   Use Lambda Proxy integration: ✅ Checked
   Lambda Region: us-east-1
   Lambda Function: enhanced-dynamodb-backup
   ```

![img](/images/3.svlessimp/api2.png)

**Resource 2: /recovery**
1. **Create resource**: `/recovery`
2. **Create method**: **POST**
3. **Integration**: `enhanced-recovery-orchestrator`

**Resource 3: /status**
1. **Create resource**: `/status`
2. **Create method**: **GET**
3. **Integration**: Create a simple status function or use mock integration

![img](/images/3.svlessimp/api3.png)

### Deploy API

1. **Actions** → **Deploy API**
2. **Deployment Configuration**:
   ```
   Deployment stage: dev
   Stage description: Development stage for backup API
   ```

![img](/images/3.svlessimp/api4.png)