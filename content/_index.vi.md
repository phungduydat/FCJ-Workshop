---
title : "Quản lý Phiên Làm Việc (Session Management)"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
---
# Làm việc với Amazon Systems Manager - Session Manager

### Tổng quan

Amazon Systems Manager – Session Manager là một dịch vụ được AWS quản lý toàn phần, cho phép truy cập shell hoặc CLI an toàn, có khả năng ghi lại và theo dõi, trực tiếp từ trình duyệt đến các phiên bản Amazon EC2 và các tài nguyên AWS khác mà **không cần mở cổng inbound** (như SSH hoặc RDP).

Trong bài lab này, bạn sẽ:

- Tìm hiểu các khái niệm cốt lõi của Session Manager và vai trò của nó trong quản trị hệ thống an toàn.
- Thực hành kết nối tới các EC2 instance **public** và **private** trong VPC mà không cần sử dụng bastion host.
- Khám phá các tính năng ghi log và kiểm toán tích hợp với **AWS CloudTrail** và **Amazon S3** để đảm bảo tuân thủ.
- Hiểu cách Session Manager giúp cải thiện bảo mật bằng cách loại bỏ kết nối inbound trực tiếp và tập trung hóa kiểm soát truy cập.

Bài thực hành này sẽ giúp các nhóm DevOps giảm thiểu rủi ro vận hành, đơn giản hóa việc quản lý truy cập hạ tầng, và nâng cao khả năng giám sát bảo mật tổng thể.

![ConnectPrivate](/images/Picture1.png) 

### Nội dung

1. [Giới thiệu](1-introduce/)
2. [Chuẩn bị Môi trường AWS](2-Prerequiste/)
3. [Triển khai Giám sát Hệ thống](3-monitoring/)
4. [Triển khai Cảnh báo & Phản hồi Tự động](4-anomaly-detection/)
5. [Phản hồi Tự động](5-automated-response/)
6. [Phát triển Dashboard](7-dashboard/)
7. [Quy trình Vận hành](8-operations/)
8. [Dọn dẹp Tài nguyên](11-cleanup/)

Mình đã chỉnh lại mục lục để thống nhất cấu trúc và cách đặt tên, đồng thời giữ đúng thứ tự logic:

---