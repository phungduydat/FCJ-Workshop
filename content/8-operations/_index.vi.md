---
title: "Operational Procedures"
date: "`r Sys.Date()`"
weight: 7
chapter: false
pre: "<b>7. </b>"
---

# ✅ Quy trình Vận hành & Tối ưu hóa (Operational Procedures & Optimization)

## 🔹 1. Thiết lập Runbook chuyên sâu

**Runbook** là tài liệu hướng dẫn chi tiết từng bước xử lý cho các kịch bản sự cố phổ biến, bao gồm:

| Kịch bản sự cố           | Dấu hiệu nhận biết                            | Nguyên nhân phổ biến                     | Bước xử lý chi tiết                                                                 | Người chịu trách nhiệm |
|--------------------------|-----------------------------------------------|------------------------------------------|--------------------------------------------------------------------------------------|--------------------------|
| CPU sử dụng 100%         | Alarm CloudWatch “HighCPUUsage”              | Load đột biến từ người dùng              | 1. Xác minh log ứng dụng  <br> 2. Kiểm tra container ECS đang chạy <br> 3. Scale ECS service thêm 1 task | DevOps Engineer         |
| Ứng dụng không phản hồi  | Timeout từ client, alarm “TargetResponseTimeHigh” | ECS service quá tải hoặc lỗi code        | 1. Kiểm tra log ứng dụng <br> 2. Scale service thủ công nếu cần <br> 3. Thiết lập auto scaling alarm     | DevOps Engineer         |

> 📁 **Vị trí lưu trữ đề xuất**: `/runbook/webenglish/ecs-cpu-spike.md`

---

## 🔹 2. Tích hợp AWS DevOps Guru

### 🧠 Giới thiệu

**AWS DevOps Guru** là dịch vụ sử dụng AI để phát hiện bất thường và đề xuất khắc phục cho hệ thống.

### 🎯 Lợi ích chính

- Tự động phát hiện sự cố mà không cần cấu hình rule thủ công
- Phân tích log, metrics, trace
- Gợi ý nguyên nhân và hành động khắc phục
- Tích hợp SNS để gửi cảnh báo
- Hiển thị trực quan qua bản đồ lỗi (Anomaly Map)

---

## 🔹 3. Bật DevOps Guru cho hệ thống

### 🔧 Các bước triển khai

1. Truy cập **AWS Console > DevOps Guru**
2. Chọn **Enable DevOps Guru**
3. Lựa chọn phạm vi:
   - **Analyze all AWS resources** *(theo dõi toàn bộ tài nguyên)*
   - Hoặc chọn **CloudFormation Stack cụ thể**
4. Cấu hình **SNS Topic** để nhận cảnh báo (tạo mới nếu chưa có)
5. Click **Enable**

> ⏳ DevOps Guru sẽ bắt đầu thu thập dữ liệu và cung cấp insight sau vài phút đến vài giờ.
![FWD](/images/8/1.png)
![FWD](/images/8/2.png)
![FWD](/images/8/3.png)
![FWD](/images/8/4.png)

---

## 🔹 4. Theo dõi & Nhận cảnh báo

### 🧾 Nội dung Insight bao gồm:

- Tên tài nguyên bị ảnh hưởng (ECS, RDS, Lambda…)
- Thời điểm xảy ra lỗi
- Metrics bất thường
- Đề xuất root cause
- Gợi ý hành động khắc phục

### 📩 Thiết lập SNS để nhận cảnh báo

- Tạo SNS Topic: `devops-guru-alerts`
- Thêm subscriber: email, Lambda, webhook…

---

## 🔹 5. Kết hợp các dịch vụ để tối ưu hóa

| Công cụ       | Mục đích sử dụng                                 |
|---------------|--------------------------------------------------|
| DevOps Guru   | Phát hiện và phân tích sự cố tự động             |
| SNS           | Gửi cảnh báo đến email, Slack, Lambda...        |
| CloudWatch    | Thu thập metrics, log, vẽ biểu đồ giám sát       |
| Notebook      | Truy vấn và phân tích tuỳ chỉnh nâng cao         |

---

## 📌 Gợi ý nâng cao

- Triển khai DevOps Guru để giảm thời gian điều tra lỗi
- Kết hợp Amazon QuickSight + SNS để gửi báo cáo định kỳ
- Tự động hóa xử lý lỗi bằng AWS Lambda hoặc Systems Manager

---

## 📚 Tài liệu tham khảo

- [🔗 AWS CloudWatch Log Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html)
- [🔗 Auto Scaling ECS](https://docs.aws.amazon.com/autoscaling/application/userguide/application-auto-scaling.html)
- [🔗 AWS DevOps Guru](https://docs.aws.amazon.com/devops-guru/latest/userguide/what-is-devops-guru.html)
- [🔗 AWS Runbook Template](https://aws.amazon.com/blogs/devops/using-aws-systems-manager-to-create-runbooks/)
