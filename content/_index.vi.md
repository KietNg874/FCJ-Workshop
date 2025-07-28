---
title : "Sao lưu & Khôi phục Thảm họa Serverless"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
---
# Workshop Sao lưu & Khôi phục Thảm họa Serverless

### Tổng quan
Trong workshop toàn diện này, bạn sẽ xây dựng một giải pháp sao lưu serverless sẵn sàng cho sản xuất, tận dụng các công nghệ serverless của AWS để cung cấp bảo vệ dữ liệu tự động, có thể mở rộng và hiệu quả về chi phí. Học cách triển khai các chiến lược khôi phục thảm họa sử dụng AWS Lambda, Step Functions, S3, DynamoDB và các dịch vụ được quản lý khác.

![Kiến trúc Sao lưu Serverless](/images/backup-architecture.jpg) 

### Những gì bạn sẽ học
- Xây dựng các giải pháp sao lưu serverless sử dụng AWS Lambda và Step Functions
- Triển khai khôi phục thảm họa đa vùng với S3 replication
- Tạo quy trình sao lưu tự động với lập lịch EventBridge
- Phát triển các quy trình xác thực dữ liệu và kiểm tra tính toàn vẹn
- Thiết lập giám sát và cảnh báo toàn diện với CloudWatch và SNS
- Thiết kế RESTful APIs cho các hoạt động sao lưu sử dụng API Gateway
- Áp dụng các thực hành bảo mật tốt nhất với IAM roles và mã hóa

### Kết quả mong đợi
Khi kết thúc workshop này, bạn sẽ đã xây dựng được:
- **3 Lambda Functions**: Các hàm sao lưu, xác thực và khôi phục
- **2 S3 Buckets**: Bucket chính và phụ với cross-region replication
- **3 DynamoDB Tables**: Dữ liệu nguồn với các bản ghi mẫu
- **1 Step Functions Workflow**: Quy trình sao lưu được điều phối
- **EventBridge Rules**: Lịch sao lưu hàng ngày và hàng tuần
- **API Gateway**: Các endpoint RESTful cho hoạt động sao lưu
- **CloudWatch Dashboard**: Giám sát và cảnh báo toàn diện

### Nội dung
 1. [Giới thiệu](1-introduce/)
 2. [Điều kiện tiên quyết](2-prerequiste/)
 3. [Triển khai Serverless](3-svlessimp/)
 4. [Kiểm thử & Xác thực](4-testing/)
 5. [Dọn dẹp](5-cleanup/)
