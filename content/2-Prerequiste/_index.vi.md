---
title : "Điều kiện tiên quyết"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

Trước khi xây dựng giải pháp sao lưu serverless, chúng ta cần thiết lập các dịch vụ AWS cơ bản và cấu hình bảo mật. Phần này sẽ hướng dẫn bạn thiết lập các thành phần hạ tầng thiết yếu mà hệ thống sao lưu của chúng ta sẽ phụ thuộc vào.

![Thiết lập Điều kiện tiên quyết](/images/2.prerequisite/001-setup.png)

## Những gì bạn sẽ tạo

Trong phần điều kiện tiên quyết này, bạn sẽ thiết lập hạ tầng cốt lõi cho giải pháp sao lưu serverless:

### Nền tảng Bảo mật
- **IAM Roles**: Vai trò dịch vụ với quyền đặc quyền tối thiểu cho các hàm Lambda, truy cập S3 và hoạt động DynamoDB
- **Quyền liên dịch vụ**: Mẫu truy cập an toàn giữa các dịch vụ AWS

### Hạ tầng Dữ liệu  
- **Bảng DynamoDB**: Ba bảng để mô phỏng dữ liệu ứng dụng thực
  - Bảng App Users (hồ sơ người dùng và tùy chọn)
  - Bảng App Orders (dữ liệu giao dịch và đơn hàng)
  - Bảng Backup Metadata (theo dõi hoạt động sao lưu)

### Hạ tầng Lưu trữ
- **S3 Bucket Chính**: Lưu trữ sao lưu chính trong vùng us-east-1
- **S3 Bucket Phụ**: Lưu trữ khôi phục thảm họa trong vùng us-west-2
- **Sao chép đa vùng**: Sao chép dữ liệu tự động cho khôi phục thảm họa

### Hệ thống Thông báo
- **SNS Topic**: Hệ thống cảnh báo cho thông báo thành công/thất bại sao lưu
- **Đăng ký email**: Thông báo thời gian thực cho các sự kiện vận hành

## Tại sao những Điều kiện tiên quyết này quan trọng

{{% notice info %}}
Những dịch vụ cơ bản này rất quan trọng cho một giải pháp sao lưu sẵn sàng sản xuất. IAM roles phù hợp đảm bảo bảo mật, bảng DynamoDB cung cấp dữ liệu thực tế để sao lưu, bucket S3 cung cấp lưu trữ bền vững, và SNS cho phép hiển thị vận hành.
{{% /notice %}}

### Thực hành Bảo mật Tốt nhất
- **Truy cập Đặc quyền Tối thiểu**: Mỗi dịch vụ chỉ nhận được quyền cần thiết
- **Bảo mật Đa vùng**: Sao chép an toàn giữa các vùng AWS
- **Dấu vết Kiểm toán**: Tất cả hành động được ghi log để tuân thủ và khắc phục sự cố

### Xuất sắc Vận hành
- **Sẵn sàng Giám sát**: Hạ tầng được chuẩn bị cho giám sát toàn diện
- **Nền tảng Có thể mở rộng**: Dịch vụ được cấu hình để xử lý tăng trưởng
- **Khôi phục Thảm họa**: Thiết lập đa vùng từ đầu

### Nội dung
2.1. [Tạo IAM Roles](2.1-roles/) \
2.2. [Tạo S3 Buckets với Cross-Region Replication](2.2-creates3bucket/) \
2.3. [Tạo DynamoDB](2.3-createdynamodb/) \
2.4. [Tạo SNS Notification](2.4-createsns/) 
