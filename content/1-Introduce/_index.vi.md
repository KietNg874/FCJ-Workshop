---
title : "Giới thiệu"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---

## Tại sao chọn Serverless cho Sao lưu & Khôi phục Thảm họa?

### Thách thức của Sao lưu Truyền thống
- **Quản lý Hạ tầng**: Duy trì các máy chủ sao lưu, hệ thống lưu trữ và mạng
- **Vấn đề Mở rộng**: Mở rộng thủ công trong thời gian cao điểm sao lưu
- **Không hiệu quả về Chi phí**: Trả tiền cho tài nguyên nhàn rỗi trong thời gian không sao lưu
- **Phức tạp**: Quản lý nhiều công cụ và hệ thống cho các loại sao lưu khác nhau
- **Chi phí Bảo trì**: Vá lỗi, cập nhật và giám sát hạ tầng sao lưu

### Ưu điểm của Serverless
- **Không cần Quản lý Hạ tầng**: Không có máy chủ để cung cấp, vá lỗi hoặc bảo trì
- **Tự động Mở rộng**: Tự động mở rộng dựa trên nhu cầu khối lượng công việc
- **Trả theo Sử dụng**: Chỉ trả tiền cho thời gian tính toán và lưu trữ thực tế được sử dụng
- **Tính khả dụng Cao**: Dự phòng tích hợp và khả năng chịu lỗi
- **Phát triển Nhanh**: Tập trung vào logic nghiệp vụ, không phải hạ tầng
- **Tích hợp**: Tích hợp tự nhiên với các dịch vụ AWS

---

## Tổng quan về Dịch vụ AWS


![Luồng dữ liệu Sao lưu Serverless](/images/001-backupdataflow.jpg)


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

## Giá trị Kinh doanh & Trường hợp Sử dụng

### Trường hợp Sử dụng Chính

#### **1. Sao lưu Dữ liệu Ứng dụng**
- **Kịch bản**: Nền tảng thương mại điện tử với dữ liệu người dùng và lịch sử đơn hàng
- **Giải pháp**: Sao lưu tự động hàng ngày các bảng DynamoDB vào S3
- **Lợi ích**: Bảo vệ dữ liệu, tuân thủ, liên tục kinh doanh

#### **2. Khôi phục Thảm họa**
- **Kịch bản**: Lỗi vùng chính ảnh hưởng đến hoạt động kinh doanh
- **Giải pháp**: Sao chép đa vùng với quy trình khôi phục tự động
- **Lợi ích**: Thời gian ngừng hoạt động tối thiểu, dự phòng địa lý

#### **3. Tuân thủ & Lưu giữ**
- **Kịch bản**: Dịch vụ tài chính yêu cầu lưu giữ dữ liệu 7 năm
- **Giải pháp**: Chính sách vòng đời tự động chuyển dữ liệu sang lưu trữ rẻ hơn
- **Lợi ích**: Tối ưu hóa chi phí, tuân thủ quy định

#### **4. Phát triển & Kiểm thử**
- **Kịch bản**: Cần bản sao dữ liệu sản xuất trong môi trường phát triển
- **Giải pháp**: Khả năng sao lưu và khôi phục điều khiển bằng API
- **Lợi ích**: Chu kỳ phát triển nhanh hơn, dữ liệu kiểm thử thực tế

### Lợi ích Kinh doanh

#### **Tối ưu hóa Chi phí**
- **Giảm Chi phí Hạ tầng**: Không có máy chủ sao lưu hoặc hệ thống lưu trữ để bảo trì
- **Mô hình Trả theo Sử dụng**: Chỉ trả tiền cho các hoạt động sao lưu thực tế
- **Tối ưu hóa Lưu trữ**: Chính sách vòng đời tự động giảm chi phí dài hạn
- **Hiệu quả Vận hành**: Giảm can thiệp thủ công và bảo trì

#### **Cải thiện Độ tin cậy**
- **Tự động Mở rộng**: Xử lý tải sao lưu thay đổi mà không cần can thiệp thủ công
- **Dự phòng Tích hợp**: Triển khai đa AZ với chuyển đổi dự phòng tự động
- **Giám sát Toàn diện**: Cảnh báo chủ động và ghi log chi tiết

#### **Tăng cường Bảo mật**
- **Mã hóa**: Dữ liệu được mã hóa khi truyền và khi nghỉ
- **Kiểm soát Truy cập**: Quyền IAM chi tiết
- **Dấu vết Kiểm toán**: Ghi log hoàn chỉnh tất cả hoạt động sao lưu
- **Tuân thủ**: Đáp ứng các yêu cầu quy định khác nhau

#### **Xuất sắc Vận hành**
- **Tự động hóa**: Cần can thiệp thủ công tối thiểu
- **Giám sát**: Dashboard và cảnh báo toàn diện
- **Khả năng mở rộng**: Tự động xử lý tăng trưởng khối lượng dữ liệu
- **Tích hợp**: Tích hợp dễ dàng với hệ thống hiện có qua API

---

### Luồng Dữ liệu

1. **Kích hoạt theo Lịch**: EventBridge kích hoạt quy trình Step Functions
2. **Điều phối Quy trình**: Step Functions phối hợp quy trình sao lưu
3. **Trích xuất Dữ liệu**: Hàm Lambda quét các bảng DynamoDB
4. **Xử lý Dữ liệu**: Xác thực, nén và định dạng
5. **Lưu trữ**: Tệp sao lưu được lưu trữ trong S3 với mã hóa
6. **Sao chép**: Sao chép đa vùng vào bucket phụ
7. **Xác thực**: Kiểm tra tính toàn vẹn và quy trình xác thực
8. **Thông báo**: Thông báo thành công/thất bại qua SNS
9. **Giám sát**: Metrics và logs được gửi đến CloudWatch

### Thành phần Chính

#### **Pipeline Sao lưu**
- **Nguồn**: Bảng DynamoDB với dữ liệu ứng dụng
- **Xử lý**: Hàm Lambda để trích xuất và xác thực dữ liệu
- **Lưu trữ**: Bucket S3 với chính sách vòng đời
- **Điều phối**: Step Functions để quản lý quy trình

#### **Khôi phục Thảm họa**
- **Sao chép**: Sao chép S3 đa vùng
- **Khôi phục**: Quy trình khôi phục tự động
- **Xác thực**: Xác minh tính toàn vẹn dữ liệu
- **Giám sát**: Thời gian khôi phục và metrics thành công

#### **Vận hành**
- **Lập lịch**: Sao lưu tự động hàng ngày và hàng tuần
- **Giám sát**: Dashboard thời gian thực và cảnh báo
- **Truy cập API**: Endpoint RESTful cho tích hợp bên ngoài
- **Bảo mật**: Mã hóa và kiểm soát truy cập xuyên suốt

---

### Metrics Hiệu suất

#### **Hiệu suất Sao lưu**
- **Thời gian Sao lưu**: < 5 phút cho bộ dữ liệu mẫu
- **Tỷ lệ Thành công**: > 95% độ tin cậy
- **Tính toàn vẹn Dữ liệu**: 100% thành công xác thực
- **Hiệu quả Lưu trữ**: Tối ưu hóa với chính sách vòng đời

#### **Hiệu suất Khôi phục**
- **Mục tiêu Thời gian Khôi phục (RTO)**: < 15 phút
- **Mục tiêu Điểm Khôi phục (RPO)**: < 24 giờ
- **Độ chính xác Dữ liệu**: 100% xác thực tính toàn vẹn
- **Chuyển đổi dự phòng Đa vùng**: < 5 phút

#### **Hiệu quả Chi phí**
- **Chi phí Workshop**: $8-12 tổng cộng
- **Vận hành Hàng tháng**: $5-8 liên tục
- **Tối ưu hóa Lưu trữ**: Giảm chi phí 60-80% với chính sách vòng đời
- **Hiệu quả Tính toán**: Mô hình serverless trả theo sử dụng

---
