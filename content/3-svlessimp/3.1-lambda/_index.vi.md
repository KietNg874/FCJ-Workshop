---
title : "Lambda Functions"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

Các hàm Lambda là thành phần xử lý cốt lõi của giải pháp sao lưu serverless. Trong phần này, bạn sẽ tạo ba hàm chuyên biệt xử lý các khía cạnh khác nhau của quy trình sao lưu và khôi phục.

![Tổng quan Lambda Functions](/images/3.svlessimp/001-overview.png)

## Tổng quan Kiến trúc Function

Giải pháp sao lưu serverless của chúng ta sử dụng ba hàm Lambda, mỗi hàm có trách nhiệm cụ thể trong vòng đời sao lưu:

### 🔄 Backup Function
**Mục đích**: Trích xuất và lưu trữ dữ liệu từ bảng DynamoDB
- **Quét bảng DynamoDB** để lấy tất cả bản ghi
- **Xuất dữ liệu ra S3** ở định dạng JSON với timestamps
- **Tạo checksums** để xác minh tính toàn vẹn dữ liệu
- **Cập nhật metadata sao lưu** với chi tiết hoạt động

### ✅ Validation Function  
**Mục đích**: Xác minh tính toàn vẹn và đầy đủ của sao lưu
- **Xác thực checksums** để đảm bảo dữ liệu không bị hỏng
- **Xác minh tính đầy đủ của file** bằng cách kiểm tra số lượng bản ghi
- **Cập nhật trạng thái sao lưu** trong bảng metadata
- **Kích hoạt cảnh báo** cho bất kỳ lỗi xác thực nào

### 🔧 Recovery Function
**Mục đích**: Khôi phục dữ liệu từ sao lưu khi cần thiết
- **Đọc file sao lưu** từ lưu trữ S3
- **Xác thực tính toàn vẹn dữ liệu** trước khi khôi phục
- **Điền vào bảng DynamoDB** với dữ liệu được khôi phục
- **Xác nhận khôi phục thành công** với kiểm tra xác minh

## Tại sao Kiến trúc này hoạt động

{{% notice info %}}
Mỗi hàm Lambda có một trách nhiệm duy nhất, làm cho hệ thống dễ bảo trì, kiểm thử và mở rộng hơn. Sự tách biệt này cho phép mở rộng độc lập và khắc phục sự cố dễ dàng hơn.
{{% /notice %}}

### Lợi ích Chính

#### **Tách biệt Mối quan tâm**
- Mỗi hàm tập trung vào một nhiệm vụ cụ thể
- Dễ dàng debug và bảo trì các thành phần riêng lẻ
- Mở rộng độc lập dựa trên yêu cầu khối lượng công việc

#### **Cô lập Lỗi**
- Lỗi trong một hàm không ảnh hưởng đến các hàm khác
- Xử lý lỗi và logic thử lại chi tiết
- Giám sát và cảnh báo cụ thể cho từng hàm

#### **Linh hoạt Vận hành**
- Các hàm có thể được gọi độc lập để kiểm thử
- Lịch thực thi khác nhau cho các hoạt động khác nhau
- Dễ dàng thêm hàm mới mà không ảnh hưởng đến các hàm hiện có

### Nội dung
1. [Backup Function](3.1.1-backupfunc/) 
2. [Validation Function](3.1.2-backupvalidator/) 
3. [Recovery Function](3.1.3-recoveryfunc/) 
