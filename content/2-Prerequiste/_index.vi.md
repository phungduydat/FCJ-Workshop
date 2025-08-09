---
title: "Các bước Chuẩn bị Môi trường AWS"
date:  "`r Sys.Date()`"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

{{% notice info %}}
Để triển khai ứng dụng container hóa lên AWS, bạn cần thiết lập một môi trường bao gồm EC2 để chạy Docker, liên kết với Amazon ECR để lưu trữ image, và sử dụng AWS CDK để tự động hóa quy trình triển khai. Cuối cùng, bạn có thể kiểm tra ứng dụng trực tiếp trên trình duyệt.
{{% /notice %}}

> 💡 Nếu bạn chưa quen với các dịch vụ trong hướng dẫn này, hãy xem thêm:

- [Giới thiệu về Amazon EC2](https://000004.awsstudygroup.com/vi/)
- [Tổng quan về Amazon Elastic Container Registry (ECR)](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)
- [AWS Cloud Development Kit (CDK)](https://docs.aws.amazon.com/cdk/latest/guide/home.html)

---

## 🚀 Nội dung Thực hiện

Trong phần này, bạn sẽ từng bước xây dựng môi trường triển khai hoàn chỉnh:

1. [Khởi tạo EC2 và cài đặt Docker](2.1-create-ec2-docker/)  
   → Tạo instance EC2, thiết lập môi trường cần thiết để chạy container.

2. [Build image và đưa lên Amazon ECR](2.2-upload-to-ecr/)  
   → Đăng nhập ECR, build Docker image và đẩy lên registry an toàn.

3. [Triển khai dự án bằng AWS CDK](2.3-deploy-cdk/)  
   → Viết mã hạ tầng dưới dạng code (IaC) để tự động hóa triển khai.

4. [Truy cập ứng dụng từ trình duyệt](2.4-access-web/)  
   → Kiểm tra kết quả triển khai qua giao diện web.

---

👉 Sau khi hoàn thành các bước này, bạn sẽ có một môi trường sẵn sàng để phát triển, triển khai và kiểm thử ứng dụng container hóa trên nền tảng AWS.

