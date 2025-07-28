---
title : "Step Function Test"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 4.2. </b> "
---

### Objectives
- Test Step Functions workflow end-to-end

---

**Lưu ý**: Đây là bản dịch tự động từ nội dung gốc. Vui lòng tham khảo file gốc để biết chi tiết đầy đủ.

**Note**: This is an automated translation from the original content. Please refer to the original file for complete details.

- Validate workflow orchestration and error handling
- Verify integration between components

### Test Step Functions Workflow

1. **Navigate to Step Functions Console**
2. **Select** `enhanced-backup-orchestration` **state machine**
3. **Start execution**:
   ```
   Execution name: integration-test-1
   Input:
   ```
   ```json
   {
     "tables": ["app-users", "app-orders", "backup-metadata"]
   }
   ```

![img](/FCJ-Workshop/images/4.testing/TEST_stepexec.png)


4. **Monitor execution**:
   - ✅ Watch visual workflow progress
   - ✅ Verify each step completes successfully
   - ✅ Check execution time < 2 minutes
   - ✅ Verify success notification received
   
![img](/FCJ-Workshop/images/4.testing/TEST_stepok.png)

### Test Workflow Error Handling

1. **Temporarily modify** backup function to cause failure:
   - Change environment variable to invalid S3 bucket
   - Or modify code to throw exception

2. **Start execution** and verify:
   - ✅ Workflow catches error
   - ✅ Failure handler executes
   - ✅ Failure notification sent
   - ✅ Execution shows failed status with details

3. **Restore** function to working state

### Test EventBridge Scheduling

1. **Temporarily modify** daily backup rule:
   - Change schedule to `cron(*/2 * * * ? *)` (every 2 minutes)
   - Save the rule

2. **Wait 2-3 minutes** and verify:
   - ✅ Step Functions execution triggered automatically
   - ✅ Backup completed successfully
   - ✅ New backup files in S3

3. **Reset schedule** to original `cron(0 2 * * ? *)`

### Test API Gateway Integration

1. **Navigate to API Gateway Console**
2. **Test /backup endpoint**:
   - Go to `/backup` → `POST` → `TEST`
   - Request body:
   ```json
   {
     "tables": ["app-users"]
   }
   ```
   - ✅ Verify 200 response
   - ✅ Check response body contains backup results

![img](/FCJ-Workshop/images/4.testing/TEST_api.png)

2. **Test /recovery endpoint**:
   - Go to `/recovery` → `POST` → `TEST`
   - Request body:
   ```json
   {
     "table_name": "app-users"
   }
   ```
   - ✅ Verify successful recovery response

![img](/FCJ-Workshop/images/4.testing/TEST_apisucc.png)

{{% notice info %}}
**Integration Testing**: This validates that all your components work together correctly. Pay attention to data flow between services and error propagation.
{{% /notice %}}
