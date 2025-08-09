---
title : "Triển khai Giám sát Hệ thống"
date :  "`r Sys.Date()`" 
weight : 3 
chapter : false
pre : " <b> 3 </b> "
---
# Hướng Dẫn Thiết Lập Hệ Thống Giám Sát Hạ Tầng WebEnglish trên AWS Linux 2

## 1. Mục tiêu

* Giám sát hiệu suất EC2 (CPU, RAM, Disk).
* Thu thập log của Spring Boot và MySQL.
* Gửi cảnh báo qua SNS.
* Hiển thị dashboard trực quan trên CloudWatch.

**Công cụ sử dụng:** AWS CloudWatch, CloudWatch Agent, Amazon SNS.

---

## 2. Yêu cầu chuẩn bị

### EC2 Instance

- Amazon Linux 2
- Đã cài đặt: Docker, Docker Compose, Java, MySQL (trong container hoặc cài trực tiếp)

### Quyền IAM Role cần có

Gán IAM role cho EC2 instance với quyền:

- `CloudWatchAgentServerPolicy`
- `AmazonSSMManagedInstanceCore`
- Quyền ghi logs và metrics vào CloudWatch

Kiểm tra role đã gán:
```bash
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
````

### Cài đặt AWS CLI (nếu chưa có)

```bash
sudo yum install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

Cấu hình CLI (thay thông tin bằng tài khoản/region của bạn):

```bash
aws configure
```

---

## 3. Cài đặt & cấu hình CloudWatch Agent

### Bước 1: Cài đặt Agent

```bash
cd /tmp
curl -O https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
sudo rpm -U ./amazon-cloudwatch-agent.rpm
```

![S3](/images/connect/1.jpg)

### Bước 2: Tạo file cấu hình

```bash
sudo nano /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
```

Dán nội dung sau:

```json
{
  "agent": {
    "metrics_collection_interval": 60,
    "logfile": "/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log",
    "run_as_user": "root"
  },
  "metrics": {
    "namespace": "WebEnglishMetrics",
    "append_dimensions": {
      "InstanceId": "${aws:InstanceId}",
      "InstanceType": "${aws:InstanceType}",
      "AutoScalingGroupName": "${aws:AutoScalingGroupName}"
    },
    "aggregation_dimensions": [["InstanceId"]],
    "metrics_collected": {
      "cpu": {
        "measurement": [
          "cpu_usage_user",
          "cpu_usage_system",
          "cpu_usage_idle"
        ],
        "metrics_collection_interval": 60,
        "totalcpu": true
      },
      "mem": {
        "measurement": ["mem_used_percent"],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": ["used_percent"],
        "metrics_collection_interval": 60,
        "resources": ["*"]
      },
      "diskio": {
        "measurement": ["io_time"],
        "metrics_collection_interval": 60
      },
      "swap": {
        "measurement": ["swap_used_percent"],
        "metrics_collection_interval": 60
      }
    }
  }
}
```

![S3](/images/4.s3/2.jpg)

### Bước 3: Khởi động Agent

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
  -s
```

![S3](/images/4.s3/3.jpg)

### Bước 4: Kiểm tra trạng thái Agent

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
```

---

## 4. Tạo cảnh báo với SNS

### Bước 1: Tạo Topic & đăng ký email

```bash
aws sns create-topic \
  --name WebEnglishAlerts \
  --region ap-northeast-1
```

Sao chép `TopicArn` từ kết quả lệnh trên.

```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:ap-northeast-1:ACCOUNT_ID:WebEnglishAlerts \
  --protocol email \
  --notification-endpoint YOUR_EMAIL@example.com \
  --region ap-northeast-1
```

![S3](/images/4.s3/5.jpg)

📧 **Xác nhận đăng ký qua email** trước khi tiếp tục.
![S3](/images/4.s3/6.jpg)

---

### Bước 2: Tạo cảnh báo CPU

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name HighCPUUsage \
  --metric-name cpu_usage_user \
  --namespace WebEnglishMetrics \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:ap-northeast-1:ACCOUNT_ID:WebEnglishAlerts \
  --region ap-northeast-1
```

---

Bạn cũng có thể gửi thử dữ liệu giả lên CloudWatch để kiểm tra:

```bash
aws cloudwatch put-metric-data \
  --metric-name cpu_usage_active \
  --namespace WebEnglishMetrics \
  --value 90
```

```

---

