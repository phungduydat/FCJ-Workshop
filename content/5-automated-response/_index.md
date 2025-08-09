---
title : "Automated Response"
date :  "`r Sys.Date()`" 
weight : 5 
chapter : false
pre : " <b> 5. </b> "
---
---

---------------------

# 🚨 Deploying a Professional Alerting & Incident Escalation System

> **Objective**: A multi-level alerting system that automatically detects anomalies, sends notifications to the right people, and responds automatically if required.

---

## 📌 1. Designing a 3-Level Alert Workflow

| Level | Name                          | Recipient        | Response Time       | Tool          |
| ----- | ----------------------------- | ---------------- | ------------------- | ------------- |
| 1     | Technical Alert (DevOps)      | DevOps Team      | ≤ 15 minutes        | Email / SNS   |
| 2     | Urgent Alert (On-call)        | On-call Dev      | ≤ 5 minutes         | SNS + Lambda  |
| 3     | Manager Alert                 | Senior Manager   | During office hours | Email / SMS   |

---

## 🧪 2. Create SNS Topics and Subscriptions

### 2.1 Create SNS Topics

```bash
aws sns create-topic --name WebEnglishAlert-Level1
aws sns create-topic --name WebEnglishAlert-Level2
aws sns create-topic --name WebEnglishAlert-Level3
````

![FWD](/images/5.fwd/1.jpg)


### 2.2 Subscribe Emails to Topics (must confirm)

```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:WebEnglishAlert-Level1 \
  --protocol email \
  --notification-endpoint devops@example.com

aws sns subscribe \
  --topic-arn arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:WebEnglishAlert-Level3 \
  --protocol email \
  --notification-endpoint ceo@example.com
```

![FWD](/images/5.fwd/2.jpg)

> ✅ **Check your email to confirm the subscription.**
> ![FWD](/images/5.fwd/3.jpg)
> ![FWD](/images/5.fwd/4.jpg)

---

## 📈 3. Create CloudWatch Alarm - Level 1

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name CPU-High-Level1 \
  --alarm-description "High CPU Alert - Level 1" \
  --metric-name cpu_usage_active \
  --namespace WebEnglishMetrics \
  --statistic Average \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --actions-enabled \
  --alarm-actions arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:WebEnglishAlert-Level1
```

![FWD](/images/5.fwd/6.jpg)

---

## 🛠 4. Escalation Lambda

### 4.1 Create IAM Role for Lambda

**trust-policy.json**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "lambda.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
```

```bash
aws iam create-role \
  --role-name LambdaAlertEscalatorRole \
  --assume-role-policy-document file://trust-policy.json

aws iam attach-role-policy \
  --role-name LambdaAlertEscalatorRole \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchReadOnlyAccess

aws iam attach-role-policy \
  --role-name LambdaAlertEscalatorRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonSNSFullAccess
```

![FWD](/images/5.fwd/7.jpg)
![FWD](/images/5.fwd/8.jpg)

---

### 4.2 Lambda Code: `alert_escalator.py`

```python
import boto3

def lambda_handler(event, context):
    cloudwatch = boto3.client('cloudwatch')
    sns = boto3.client('sns')

    alarm_name = "CPU-High-Level1"
    response = cloudwatch.describe_alarms(AlarmNames=[alarm_name])

    if response['MetricAlarms'] and response['MetricAlarms'][0]['StateValue'] == "ALARM":
        sns.publish(
            TopicArn="arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:WebEnglishAlert-Level2",
            Subject="🚨 Escalation Alert",
            Message=f"Alarm {alarm_name} has not been handled. Escalating to Level 2."
        )
```

---

### 4.3 Deploy Lambda Function

```bash
zip function.zip alert_escalator.py

aws lambda create-function \
  --function-name alertEscalator \
  --runtime python3.12 \
  --role arn:aws:iam::<ACCOUNT_ID>:role/LambdaAlertEscalatorRole \
  --handler alert_escalator.lambda_handler \
  --zip-file fileb://function.zip \
  --timeout 10
```

> ✅ You can test it in AWS Console → Lambda → Test
> ![FWD](/images/5.fwd/7.jpg)

---

## 📈 5. Create Alarm Triggering Lambda - Level 2

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name CPU-High-Level2 \
  --alarm-description "CPU high alert escalation" \
  --metric-name cpu_usage_active \
  --namespace WebEnglishMetrics \
  --statistic Average \
  --period 600 \
  --evaluation-periods 2 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --actions-enabled \
  --alarm-actions arn:aws:lambda:ap-northeast-1:<ACCOUNT_ID>:function:alertEscalator
```

---

## 🔁 Escalation Overview

| Level | Alarm           | Condition          | Action                                              |
| ----- | --------------- | ------------------ | --------------------------------------------------- |
| 1     | CPU-High-Level1 | 1 × 5 min > 80%    | Send SNS Level 1 → DevOps                           |
| 2     | CPU-High-Level2 | 2 × 10 min > 80%   | Call Lambda → send SNS Level 2 (notify on-call Dev) |
| 3     | (optional)      | No response in 30m | Send SNS Level 3 → senior management via email/SMS  |

---

## 📊 6. CloudWatch Anomaly Detection (Advanced Option)

```bash
aws cloudwatch put-anomaly-detector \
  --namespace WebEnglishMetrics \
  --metric-name cpu_usage_active \
  --statistic Average \
  --dimensions Name=InstanceId,Value=i-xxxxxx
```

![FWD](/images/5.fwd/8.jpg)

---

## 🌀 7. Step Functions (Optional Automated Response)

Use to:

* Scale ECS Service
* Reboot EC2 Instance
* Trigger a sequence of actions when an incident is detected

---

## 🧪 8. System Testing

```bash
sudo yum install -y stress-ng
stress-ng --cpu 4 --timeout 600s
```

You can also push fake metrics to CloudWatch for testing:

```bash
aws cloudwatch put-metric-data \
  --metric-name cpu_usage_active \
  --namespace WebEnglishMetrics \
  --value 90
```

Check:

* Is SNS sending to the correct recipients?
* Was Lambda triggered?
* Was escalation executed?
* Were email confirmations completed?

---

## 📘 9. Summary

✅ Multi-level alerting system works as expected:

* CloudWatch monitors & detects anomalies
* SNS routes alerts to the correct recipients
* Lambda responds if alerts are not handled
* Extendable with Step Functions, PagerDuty, EC2 auto recovery...
  ![FWD](/images/5.fwd/9.jpg)
  ![FWD](/images/5.fwd/10.jpg)

---

## 🧩 Advanced Suggestions

* Use **CloudFormation / CDK** to define the entire infrastructure as code
* Log into **CloudWatch Logs** for escalation tracking
* Create a **Dead Letter Queue** for Lambda in case of failures

```

---

Nếu bạn muốn, mình có thể **đưa toàn bộ bản tiếng Anh này thành file `.md`** để bạn thêm trực tiếp vào hệ thống Hugo hoặc GitBook.  
Bạn có muốn mình tạo file đó luôn không?
```
