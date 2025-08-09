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
2. [Yêu cầu chuẩn bị](2-Prerequiste/)
3. [Triển khai Giám sát](3-monitoring/)
4. [Phát hiện Bất thường](4-anomaly-detection/)
5. [Phản hồi Tự động](5-automated-response/)
6. [Quy trình Leo thang Sự cố](6-escalation/)
7. [Phát triển Dashboard](7-dashboard/)
8. [Quy trình Vận hành](8-operations/)
<!-- 9. [Tối ưu Hiệu năng](9-performance/)
10. [Điều chỉnh Cảnh báo](10-alert-tuning/) -->
11. [Dọn dẹp Tài nguyên](11-cleanup/)
