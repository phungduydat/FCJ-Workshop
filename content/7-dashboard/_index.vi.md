---
title : "Dashboard Development"
date :  "`r Sys.Date()`" 
weight : 6 
chapter : false
pre : " <b> 6. </b> "
---

# 📊 Triển Khai Dashboard và Báo Cáo Trực Quan Cho Dự Án WebEnglish

Tài liệu này cung cấp quy trình chuyên nghiệp và chi tiết để thiết lập các bảng điều khiển (dashboards) và báo cáo giám sát dữ liệu hệ thống trong dự án WebEnglish. Việc giám sát được thực hiện bằng Amazon CloudWatch và Amazon QuickSight để cung cấp khả năng hiển thị thời gian thực cũng như các phân tích chiến lược giúp đội DevOps phản ứng nhanh chóng với sự cố sản phẩm.

---

## 🎯 Mục Tiêu

- Hiển thị hiệu suất hệ thống EC2, container, ứng dụng Spring Boot, và MySQL.
- Cảnh báo khi có dấu hiệu bất thường dựa trên hành vi (anomaly).
- Cung cấp báo cáo trực quan định kỳ cho DevOps và quản lý cấp cao.

---

## 🧭 Bước 1: Truy cập CloudWatch Dashboard
- Vào AWS Console → Tìm “CloudWatch”.
- Vào **Dashboards** → Chọn hoặc tạo dashboard mới.
![FWD](/images/9/2.jpg)
![FWD](/images/9/3.jpg)

## 📊 Bước 2: Thêm metric biểu đồ
- Click **“Add widget”** → Chọn **Line** → Click **Next**.
- Trong phần “Browse” chọn:
  - Region: Tokyo (hoặc nơi instance đang chạy)
  - Namespace: `WebEnglishMetrics` (theo bạn cấu hình)
  - Metric: `cpu_usage_active`
![FWD](/images/9/4.jpg)
![FWD](/images/9/5.jpg)
![FWD](/images/9/6.jpg)

## 📈 Bước 3: Thêm Anomaly Detection Band
- Trong phần **Graphed metrics**, click **“Add math”** → chọn `ANOMALY_DETECTION_BAND(m1, 2)`
  - `m1` là ID metric gốc.
  - `2` là độ nhạy (sensitivity).
![FWD](/images/9/7.jpg)
![FWD](/images/9/8.jpg)

## 🚨 Bước 4: Tạo Alarm từ metric
- Vào tab **Actions** → Click **“Create Alarm”**
- Chọn metric `cpu_usage_active`
- Chọn kiểu thống kê: `Average`
- Period: `5 minutes`
- **Conditions**:
  - Threshold type: `Anomaly detection`
  - Anomaly detection model: chọn dòng `ANOMALY_DETECTION_BAND(m1, 2)`
![FWD](/images/9/9.jpg)
![FWD](/images/9/10.jpg)

## ⚙️ Bước 5: Additional configuration

| Tùy chọn | Gợi ý cấu hình |
|---------|----------------|
| **Datapoints to alarm** | `3 out of 5` |
| **Missing data treatment** | `Treat missing data as missing` |
| **Actions suppression** | Bỏ qua nếu không dùng rule khác |

## 🏁 Bước 6: Đặt tên & tạo Alarm
- Đặt tên báo động: ví dụ `HighCPUWithAnomalyDetection`
- Chọn SNS topic nếu muốn gửi email
- Click **Create Alarm**
---

## 🧾 Mẹo Vận Hành

- ⏱️ Giới hạn thời gian giữ logs: `7–14 ngày` để giảm chi phí.
- 🔔 Cấu hình gửi cảnh báo qua **SNS Topic** tới email hoặc Lambda handler.
- 🧪 Kết hợp với **AWS DevOps Guru** để tự động phát hiện root cause.

---

## 📚 Tài Liệu Tham Khảo

- [CloudWatch Dashboards](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html)
- [Amazon QuickSight Getting Started](https://docs.aws.amazon.com/quicksight/latest/user/welcome.html)
- [Anomaly Detection](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Anomaly_Detection.html)
