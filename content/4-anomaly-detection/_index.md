---
title: "Deploying Automatic Alerting and Response for WebEnglish"
date :  "`r Sys.Date()`" 
weight : 4 
chapter : false
pre : " <b> 4. </b> "
---
-------------------

# 🚨 Implementing Automatic Alerts and Response with CloudWatch, SNS, and Lambda

This document provides step-by-step guidance on integrating Amazon CloudWatch, SNS, Lambda, and optionally DevOps Guru to detect anomalies and automatically respond in the WebEnglish system.

---
## 🎯 Objectives

  * Detect performance anomalies in EC2 or ECS.
  * Send alerts via email and trigger Lambda for automated response.
  * Automatically adjust the desired number of **ECS** service tasks or restart instances.

-----

## ✅ 1. Create a CloudWatch Anomaly Detector

Create an anomaly detector for the `cpu_usage_active` metric in the **namespace** `WebEnglishMetrics`.

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

## ✅ 2. Create an SNS Topic and Subscribe an Email

Create an SNS topic for sending alerts and subscribe an email endpoint.

```bash
aws sns create-topic --name WebEnglishAlerts --region ap-northeast-1
aws sns subscribe \
--topic-arn arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:WebEnglishAlerts \
--protocol email \
--notification-endpoint your-email@example.com \
--region ap-northeast-1
```

> 📩 Don’t forget to confirm the subscription via the email inbox.
![S3](/images/4.s3/2.jpg)
![S3](/images/4.s3/3.jpg)
![S3](/images/4.s3/4.jpg)

-----

## ✅ 3. Create a CloudWatch Alarm from the Anomaly Detector

Set up an alarm to trigger when the `cpu_usage_active` metric exceeds the anomaly threshold.

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

## ✅ 4. Create a Lambda Function to Scale ECS

### A. Create an IAM Role

Create the IAM role `LambdaScaleECSRole` with a trust policy for Lambda.

```bash
aws iam create-role \
--role-name LambdaScaleECSRole \
--assume-role-policy-document file://trust-policy.json
```

**trust-policy.json**:

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

### B. Attach Required Policies

Grant permissions for logging and ECS interactions.

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

## ✅ 5. Create the Lambda Script `scale_ecs_service.py`

This Python function updates the ECS service desired task count.

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

**Zip the file:**

```bash
zip function.zip scale_ecs_service.py
```

-----

## ✅ 6. Deploy the Lambda Function

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

## ✅ 7. Link Lambda to SNS and Grant Permission

Subscribe the Lambda function to the SNS topic.

```bash
aws sns subscribe \
--topic-arn arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:WebEnglishAlerts \
--protocol lambda \
--notification-endpoint arn:aws:lambda:ap-northeast-1:<ACCOUNT_ID>:function:ScaleWebEnglish \
--region ap-northeast-1
```
![S3](/images/4.s3/12.jpg)

**Grant SNS invoke permissions:**

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

## ✅ 8. (Optional) Create a Step Function for Scaling ECS

Use AWS Step Functions to orchestrate scaling.

**scale-ecs-step.json**:

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

### Create Step Function Role

```bash
aws iam create-role \
--role-name StepFunctionExecutionRole \
--assume-role-policy-document file://stepfunction-trust-policy.json
aws iam attach-role-policy \
--role-name StepFunctionExecutionRole \
--policy-arn arn:aws:iam::aws:policy/AmazonECS_FullAccess
```

### Create the State Machine

```bash
aws stepfunctions create-state-machine \
--name ScaleWebEnglishStepFunction \
--definition file://scale-ecs-step.json \
--role-arn arn:aws:iam::<ACCOUNT_ID>:role/StepFunctionExecutionRole \
--type STANDARD \
--region ap-northeast-1
```

-----

## ✅ 9. Test the System

| Goal | Test Method |
| :--- | :--- |
| **Increase CPU** | `stress-ng` or Spring Boot endpoint |
| **Lambda Logs** | `CloudWatch Logs` → `/aws/lambda/ScaleWebEnglish` |
| **Scale ECS** | Console → `WebenglishService` |
| **SNS Functionality** | Publish test message via CLI |

![S3](/images/4.s3/15.jpg)

**Increase CPU (Ubuntu):**

```bash
sudo apt update && sudo apt install stress-ng -y
stress-ng --cpu 2 --timeout 300s
```

**Or Spring Boot endpoint:**

```java
@GetMapping("/load-cpu")
public String loadCpu() {
    while (true) {
        Math.pow(Math.random(), Math.random());
    }
}
```

**Invoke endpoint:**

```bash
curl http://your-ecs-app/load-cpu
```

**Test SNS → Lambda:**

```bash
aws sns publish \
--topic-arn arn:aws:sns:ap-northeast-1:<ACCOUNT_ID>:WebEnglishAlerts \
--message '{"test": "SNS to Lambda"}' \
--region ap-northeast-1
```
![S3](/images/4.s3/16.jpg)
![S3](/images/4.s3/17.jpg)

-----

## ✅ 10. Debug & Optimization Tips

| Component | Notes |
| :--- | :--- |
| **Lambda** | Add explicit exception handling and detailed logging. |
| **IAM Role** | Follow **Least Privilege** principle for permissions. |
| **ECS** | Consider using a Target Tracking Policy instead of manual scaling in Lambda. |
| **Alarm** | Tune `evaluation-periods` to reduce false positives. |

**📌 Manually test Lambda if needed:**

```bash
aws lambda invoke \
--function-name ScaleWebEnglish \
--payload '{}' \
output.json \
--region ap-northeast-1
```
```

---

Do you want me to now **convert this into a clean, ready-to-publish `.md` file** with all the images and formatting intact for your documentation site? That way it’s ready for Hugo or GitBook without further edits.