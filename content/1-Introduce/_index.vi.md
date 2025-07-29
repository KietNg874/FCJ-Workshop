---
title : "Giới thiệu"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---

## Tại sao chọn Serverless Backup & Disaster Recovery?

### Ưu điểm của Serverless
- **Không cần Quản lý Hạ tầng**: Không có máy chủ để cung cấp, vá lỗi hoặc bảo trì
- **Tự động Mở rộng**: Tự động mở rộng dựa trên nhu cầu khối lượng công việc
- **Trả theo Sử dụng**: Chỉ trả tiền cho thời gian tính toán và lưu trữ thực tế được sử dụng
- **Tính khả dụng Cao**: Dự phòng tích hợp và khả năng chịu lỗi
- **Phát triển Nhanh**: Tập trung vào logic nghiệp vụ, không phải hạ tầng
- **Tích hợp**: Tích hợp tự nhiên với các dịch vụ AWS

---

## Tổng quan về Dịch vụ AWS


![Luồng dữ liệu Sao lưu Serverless](/FCJ-Workshop/images/001-backupdataflow.jpg)


### Dịch vụ Tính toán Cốt lõi

#### **AWS Lambda**
- **Mục đích**: Thực thi logic sao lưu, xử lý dữ liệu và điều phối
- **Lợi ích**: 
  - Tính toán serverless với tự động mở rộng
  - Chỉ trả tiền cho thời gian thực thi
  - Khả năng chịu lỗi và ghi log tích hợp
- **Trường hợp sử dụng**: Các hàm sao lưu, xác thực dữ liệu, điều phối khôi phục

#### **AWS Step Functions**
- **Mục đích**: Điều phối các quy trình sao lưu phức tạp
- **Lợi ích**:
  - Quản lý quy trình trực quan
  - Xử lý lỗi và logic thử lại
  - Quản lý trạng thái và phối hợp
- **Trường hợp sử dụng**: Quy trình sao lưu nhiều bước, quy trình khôi phục

### Dịch vụ Lưu trữ

#### **Amazon S3**
- **Mục đích**: Lưu trữ sao lưu chính với nhiều lớp lưu trữ
- **Lợi ích**:
  - Nhiều lớp lưu trữ để tối ưu hóa chi phí
  - Sao chép đa vùng cho khôi phục thảm họa
  - Chính sách vòng đời để quản lý chi phí tự động
- **Trường hợp sử dụng**: Lưu trữ tệp sao lưu, sao chép đa vùng

#### **Amazon DynamoDB**
- **Mục đích**: Lưu trữ dữ liệu nguồn và metadata sao lưu
- **Lợi ích**:
  - Cơ sở dữ liệu NoSQL được quản lý hoàn toàn
  - Tính năng bảo mật và sao lưu tích hợp
  - Bảng toàn cầu cho triển khai đa vùng
- **Trường hợp sử dụng**: Dữ liệu ứng dụng, theo dõi metadata sao lưu

### Dịch vụ Tích hợp & Giám sát

#### **Amazon EventBridge**
- **Mục đích**: Lập lịch sao lưu tự động và kích hoạt quy trình
- **Lợi ích**:
  - Bus sự kiện serverless
  - Lập lịch linh hoạt với biểu thức cron
  - Tích hợp với nhiều dịch vụ AWS
- **Trường hợp sử dụng**: Sao lưu theo lịch, quy trình điều khiển bằng sự kiện

#### **Amazon SNS**
- **Mục đích**: Thông báo và cảnh báo
- **Lợi ích**:
  - Nhắn tin đa giao thức
  - Mẫu nhắn tin fan-out
  - Tích hợp với các endpoint khác nhau
- **Trường hợp sử dụng**: Thông báo thành công/thất bại sao lưu, cảnh báo

#### **Amazon CloudWatch**
- **Mục đích**: Giám sát, ghi log và cảnh báo
- **Lợi ích**:
  - Giám sát và ghi log toàn diện
  - Metrics tùy chỉnh và dashboard
  - Khả năng cảnh báo chủ động
- **Trường hợp sử dụng**: Giám sát hiệu suất, dashboard vận hành

#### **Amazon API Gateway**
- **Mục đích**: RESTful API cho các hoạt động sao lưu
- **Lợi ích**:
  - Dịch vụ API được quản lý hoàn toàn
  - Bảo mật và điều tiết tích hợp
  - Tích hợp với các hàm Lambda
- **Trường hợp sử dụng**: Tích hợp hệ thống bên ngoài, kích hoạt sao lưu thủ công

### Bảo mật & Quản lý Truy cập

#### **AWS IAM**
- **Mục đích**: Bảo mật và kiểm soát truy cập
- **Lợi ích**:
  - Quyền chi tiết
  - Kiểm soát truy cập dựa trên vai trò
  - Tích hợp với tất cả dịch vụ AWS
- **Trường hợp sử dụng**: Quyền dịch vụ, truy cập đặc quyền tối thiểu

---
