---
title : "Recovery Simulation"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 4.3. </b> "
---

### Xác minh Cross-Region Replication

1. **Kiểm tra primary bucket** (us-east-1):
   - Ghi chú các backup files hiện tại và số lượng
   - Upload một test file để xác minh replication

2. **Kiểm tra secondary bucket** (us-west-2):
   - Chuyển AWS Console region sang us-west-2
   - Điều hướng đến secondary S3 bucket
   - ✅ Xác minh tất cả backup files được replicated
   - ✅ Xác minh test file được replicated trong vòng 5 phút

### Mô phỏng Primary Region Failure

**Kịch bản**: Primary region trở nên không khả dụng

1. **Ghi lại trạng thái hiện tại**:
   - Đếm items trong mỗi DynamoDB table
   - Ghi chú timestamps backup mới nhất
   - Ghi lại checksums dữ liệu hiện tại

2. **Mô phỏng mất dữ liệu**:
   - Xóa một số items từ bảng `app-users`
   - Ghi lại những gì đã xóa để validation

3. **Recovery từ secondary region**:
   - Chuyển sang vùng us-west-2
   - Truy cập các replicated backup files
   - Sử dụng recovery function với secondary bucket

### Test Recovery Process

1. **Tạo temporary recovery function** trong us-west-2:
   - Copy `enhanced-recovery-orchestrator` sang us-west-2
   - Cập nhật environment variables để sử dụng secondary bucket

2. **Thực thi recovery**:
   ```json
   {
     "table_name": "app-users",
     "backup_bucket": "serverless-backup-secondary-[yourname]"
   }
   ```

3. **Validate recovery**:
   - ✅ Xác minh các items đã xóa được khôi phục
   - ✅ Kiểm tra tính toàn vẹn dữ liệu với checksums
   - ✅ Xác nhận số lượng items khớp với trạng thái trước lỗi

### Tính toán Recovery Metrics

**Recovery Time Objective (RTO)**:
- Thời gian từ phát hiện lỗi đến khôi phục dịch vụ
- Mục tiêu: < 15 phút
- ✅ Đo thời gian thực tế

**Recovery Point Objective (RPO)**:
- Mức mất dữ liệu tối đa có thể chấp nhận
- Mục tiêu: < 24 giờ (backup cuối cùng)
- ✅ Xác minh không mất dữ liệu ngoài backup cuối

{{% notice tip %}}
**Disaster Recovery Testing**: Kiểm thử DR thường xuyên là rất quan trọng. Trong production, bạn nên kiểm thử các quy trình khôi phục thảm họa ít nhất mỗi quý.
{{% /notice %}}

### Chi tiết Quy trình Recovery Simulation

#### **Bước 1: Chuẩn bị Baseline**

**Ghi lại trạng thái ban đầu:**
```bash
# Đếm items trong các tables
aws dynamodb scan --table-name app-users --select COUNT --region us-east-1
aws dynamodb scan --table-name app-orders --select COUNT --region us-east-1
aws dynamodb scan --table-name backup-metadata --select COUNT --region us-east-1
```

**Kiểm tra backup files:**
```bash
# Liệt kê backup files trong primary bucket
aws s3 ls s3://serverless-backup-primary-yourname/dynamodb-backups/ --region us-east-1

# Kiểm tra replication trong secondary bucket  
aws s3 ls s3://serverless-backup-secondary-yourname/dynamodb-backups/ --region us-west-2
```

#### **Bước 2: Mô phỏng Data Loss**

**Xóa dữ liệu test:**
1. Truy cập DynamoDB Console
2. Mở bảng `app-users`
3. Xóa 1-2 items (ghi lại userId để validation sau)
4. Ghi chú thời gian xóa

**Ví dụ items bị xóa:**
- userId: "user001"
- userId: "user002"

#### **Bước 3: Cross-Region Recovery**

**Tạo recovery function trong us-west-2:**
1. Chuyển Console region sang us-west-2
2. Tạo Lambda function mới: `disaster-recovery-function`
3. Copy code từ `enhanced-recovery-orchestrator`
4. Cập nhật environment variables:
   ```
   BACKUP_BUCKET = serverless-backup-secondary-yourname
   ```

**Thực thi recovery:**
```json
{
  "table_name": "app-users",
  "backup_bucket": "serverless-backup-secondary-yourname",
  "batch_size": 25
}
```

#### **Bước 4: Validation**

**Kiểm tra recovery thành công:**
1. Đếm lại items trong bảng `app-users`
2. Xác minh các items đã xóa được khôi phục
3. So sánh checksums trước và sau recovery

**Expected Results:**
```json
{
  "statusCode": 200,
  "body": {
    "message": "Recovery hoàn thành thành công",
    "table_name": "app-users", 
    "items_restored": 3,
    "validation_passed": true
  }
}
```

#### **Bước 5: Metrics Calculation**

**RTO Measurement:**
- Thời gian phát hiện lỗi: T1
- Thời gian bắt đầu recovery: T2  
- Thời gian hoàn thành recovery: T3
- **RTO = T3 - T1**

**RPO Assessment:**
- Thời gian backup cuối: T_backup
- Thời gian xảy ra lỗi: T_failure
- **RPO = T_failure - T_backup**

### Kết quả Mong đợi

#### **Cross-Region Replication:**
- ✅ Tất cả backup files xuất hiện trong secondary bucket
- ✅ Replication lag < 5 phút
- ✅ File integrity được duy trì

#### **Recovery Performance:**
- ✅ RTO < 15 phút (bao gồm detection + recovery)
- ✅ RPO < 24 giờ (tần suất backup hàng ngày)
- ✅ 100% data recovery success rate

#### **Business Continuity:**
- ✅ Dịch vụ có thể được khôi phục từ secondary region
- ✅ Không mất dữ liệu quan trọng
- ✅ Quy trình recovery có thể tự động hóa

### Troubleshooting

#### **Replication Issues:**
1. **Delay in replication**: Kiểm tra S3 replication rule status
2. **Missing files**: Xác minh source bucket có files
3. **Permission errors**: Kiểm tra replication role permissions

#### **Recovery Issues:**
1. **Lambda timeout**: Tăng timeout cho recovery function
2. **DynamoDB throttling**: Giảm batch size
3. **Cross-region permissions**: Xác minh IAM roles hoạt động cross-region

#### **Validation Failures:**
1. **Checksum mismatch**: Kiểm tra data corruption
2. **Item count mismatch**: Xác minh backup completeness
3. **Key schema issues**: Đảm bảo table schema consistency
