---
title: "Triển khai Tính năng Cảnh Báo và Phản Hồi Tự Động cho WebEnglish"
date :  "`r Sys.Date()`" 
weight : 4 
chapter : false
pre : " <b> 4. </b> "
---
-------------------

# 🚨 Triển khai Cảnh Báo và Phản Hồi Tự Động với CloudWatch, SNS và Lambda

Tài liệu này hướng dẫn chi tiết cách tích hợp Amazon CloudWatch, SNS, Lambda, và (tuỳ chọn) DevOps Guru để phát hiện bất thường và tự động phản ứng trong hệ thống WebEnglish.

---
## 🎯 Mục Tiêu

  * Phát hiện bất thường hiệu suất của EC2 hoặc ECS.
  * Gửi cảnh báo qua email và kích hoạt Lambda để phản ứng.
  * Tự động điều chỉnh số lượng task của dịch vụ **ECS** hoặc khởi động lại.

-----

## ✅ 1. Tạo CloudWatch Anomaly Detector

Tạo trình phát hiện bất thường cho chỉ số `cpu_usage_active` trong **namespace** `WebEnglishMetrics`.

```bash
aws cloudwatch put-anomaly-detector \
--namespace "WebEnglishMetrics" \
--metric-name "cpu_usage_active" \
--stat "Average" \
--dimensions Name=InstanceId,Value=i-0817b4fd50252b509 \
--region ap-northeast-1
```
![S3](/images/4.s3/1.jpg)
-----

## ✅ 2. Tạo SNS Topic và Đăng ký Email

Tạo một SNS topic để gửi cảnh báo và đăng ký một email để nhận thông báo.

```bash
aws sns create-topic --name WebEnglishAlerts --region ap-northeast-1
aws sns subscribe \
--topic-arn arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:WebEnglishAlerts \
--protocol email \
--notification-endpoint your-email@example.com \
--region ap-northeast-1
```

> 📩 Đừng quên xác nhận email trong hộp thư đến để kích hoạt subscription.
![S3](/images/4.s3/2.jpg)
![S3](/images/4.s3/3.jpg)
![S3](/images/4.s3/4.jpg)

-----

## ✅ 3. Tạo CloudWatch Alarm từ Anomaly Detector

Tạo một cảnh báo để kích hoạt khi chỉ số `cpu_usage_active` vượt quá ngưỡng bất thường.

```bash
aws cloudwatch put-metric-alarm \
--alarm-name CPUAnomalyAlarm \
--evaluation-periods 2 \
--datapoints-to-alarm 2 \
--treat-missing-data notBreaching \
--comparison-operator GreaterThanUpperThreshold \
--metrics '[
{
"Id": "m1",
"MetricStat": {
"Metric": {
"Namespace": "WebEnglishMetrics",
"MetricName": "cpu_usage_active",
"Dimensions": [
{
"Name": "InstanceId",
"Value": "i-0817b4fd50252b509"
}
]
},
"Period": 60,
"Stat": "Average"
},
"ReturnData": true
},
{
"Id": "ad1",
"Expression": "ANOMALY_DETECTION_BAND(m1, 2)",
"Label": "AnomalyDetectionBand",
"ReturnData": true
}
]' \
--threshold-metric-id ad1 \
--alarm-actions arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:WebEnglishAlerts \
--region ap-northeast-1
```
![S3](/images/4.s3/5.jpg)

-----

## ✅ 4. Tạo Lambda để Scale ECS

### A. Tạo IAM Role

Tạo IAM role `LambdaScaleECSRole` với chính sách tin cậy cho Lambda.

```bash
aws iam create-role \
--role-name LambdaScaleECSRole \
--assume-role-policy-document file://trust-policy.json
```

Nội dung file `trust-policy.json`:

```json
{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Principal": {
"Service": "lambda.amazonaws.com"
},
"Action": "sts:AssumeRole"
}
]
}
```
![S3](/images/4.s3/6.jpg)
![S3](/images/4.s3/7.jpg)

![S3](/images/4.s3/8.jpg)

### B. Gắn Policy cần thiết

Gắn các chính sách cần thiết để Lambda có thể ghi log và tương tác với ECS.

```bash
aws iam attach-role-policy \
--role-name LambdaScaleECSRole \
--policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam attach-role-policy \
--role-name LambdaScaleECSRole \
--policy-arn arn:aws:iam::aws:policy/AmazonECS_FullAccess
```
![S3](/images/4.s3/9.jpg)

-----

## ✅ 5. Tạo Hàm Lambda `scale_ecs_service.py`

Hàm Python này sẽ gọi API của AWS để điều chỉnh số lượng **task** mong muốn của dịch vụ ECS.

```python
import boto3

def lambda_handler(event, context):
    ecs = boto3.client('ecs')
    cluster = 'MyCluster'
    service = 'WebenglishService'
    desired_count = 2

    try:
        response = ecs.update_service(
            cluster=cluster,
            service=service,
            desiredCount=desired_count
        )
        print("Service updated:", response['service']['serviceName'])
        return {
            'statusCode': 200,
            'body': 'Scaling executed'
        }
    except Exception as e:
        print("Error:", str(e))
        return {
            'statusCode': 500,
            'body': str(e)
        }
```
![S3](/images/4.s3/10.jpg)

**Nén file:**

```bash
zip function.zip scale_ecs_service.py
```

-----

## ✅ 6. Tạo Lambda Function

Tạo hàm Lambda có tên `ScaleWebEnglish` từ file nén vừa tạo.

```bash
aws lambda create-function \
--function-name ScaleWebEnglish \
--runtime python3.9 \
--role arn:aws:iam::<ACCOUNT_ID>:role/LambdaScaleECSRole \
--handler scale_ecs_service.lambda_handler \
--zip-file fileb://function.zip \
--timeout 10 \
--region ap-northeast-1
```
![S3](/images/4.s3/11.jpg)

-----

## ✅ 7. Gắn Lambda vào SNS và Cấp Quyền

Đăng ký Lambda vào SNS topic để nó có thể được kích hoạt khi có cảnh báo.

```bash
aws sns subscribe \
--topic-arn arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:WebEnglishAlerts \
--protocol lambda \
--notification-endpoint arn:aws:lambda:ap-northeast-1:<ACCOUNT_ID>:function:ScaleWebEnglish \
--region ap-northeast-1
```
![S3](/images/4.s3/12.jpg)

**Cấp quyền cho SNS gọi Lambda:**

```bash
aws lambda add-permission \
--function-name ScaleWebEnglish \
--statement-id snsInvokePermission \
--action lambda:InvokeFunction \
--principal sns.amazonaws.com \
--source-arn arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:WebEnglishAlerts \
--region ap-northeast-1
```
![S3](/images/4.s3/12.jpg)

-----

## ✅ 8. (Tùy chọn) Tạo Step Function cho Scale ECS

Sử dụng Step Functions để tạo luồng công việc tự động.

### File `scale-ecs-step.json`

```json
{
"Comment": "Scale ECS when high CPU",
"StartAt": "ScaleService",
"States": {
"ScaleService": {
"Type": "Task",
"Resource": "arn:aws:states:::aws-sdk:ecs:updateService",
"Parameters": {
"Cluster": "MyCluster",
"Service": "WebenglishService",
"DesiredCount": 2
},
"End": true
}
}
}
```

### Tạo role cho Step Function

```bash
aws iam create-role \
--role-name StepFunctionExecutionRole \
--assume-role-policy-document file://stepfunction-trust-policy.json
aws iam attach-role-policy \
--role-name StepFunctionExecutionRole \
--policy-arn arn:aws:iam::aws:policy/AmazonECS_FullAccess
```

### Tạo State Machine

```bash
aws stepfunctions create-state-machine \
--name ScaleWebEnglishStepFunction \
--definition file://scale-ecs-step.json \
--role-arn arn:aws:iam::<ACCOUNT_ID>:role/StepFunctionExecutionRole \
--type STANDARD \
--region ap-northeast-1
```

-----

## ✅ 9. Kiểm Thử Hệ Thống

| Mục tiêu | Cách kiểm thử |
| :--- | :--- |
| **Tăng CPU** | `stress-ng` hoặc endpoint Spring Boot |
| **Log Lambda** | `CloudWatch Logs` → `/aws/lambda/ScaleWebEnglish` |
| **Scale ECS** | Console → `WebenglishService` |
| **Hoạt động SNS** | Gửi thử message bằng CLI |

![S3](/images/4.s3/15.jpg)

### 🔹 Tăng CPU (Ubuntu):

```bash
sudo apt update && sudo apt install stress-ng -y
stress-ng --cpu 2 --timeout 300s
```

### 🔹 Hoặc endpoint Spring Boot:

```java
@GetMapping("/load-cpu")
public String loadCpu() {
    while (true) {
        Math.pow(Math.random(), Math.random());
    }
}
```

Gọi endpoint:

```bash
curl http://your-ecs-app/load-cpu
```

### 🔹 Test SNS → Lambda:

```bash
aws sns publish \
--topic-arn arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:WebEnglishAlerts \
--message '{"test": "SNS to Lambda"}' \
--region ap-northeast-1
```
![S3](/images/4.s3/16.jpg)
![S3](/images/4.s3/17.jpg)

-----

## ✅ 10. Mẹo Debug & Tối Ưu

| Thành phần | Lưu ý |
| :--- | :--- |
| **Lambda** | Thêm xử lý `exception` rõ ràng và log chi tiết. |
| **IAM Role** | Chỉ cấp quyền tối thiểu cần thiết để tuân thủ nguyên tắc **Least Privilege**. |
| **ECS** | Cân nhắc sử dụng chính sách `Target Tracking Policy` thay vì điều chỉnh thủ công trong Lambda. |
| **Alarm** | Điều chỉnh `evaluation-periods` hợp lý để tránh cảnh báo giả. |

**📌 Test thủ công Lambda nếu cần:**

```bash
aws lambda invoke \
--function-name ScaleWebEnglish \
--payload '{}' \
output.json \
--region ap-northeast-1
```