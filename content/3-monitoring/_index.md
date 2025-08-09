
---
title : "Monitoring Implementation"
date :  "`r Sys.Date()`" 
weight : 3 
chapter : false
pre : " <b> 3 </b> "
---
# Guide to Setting Up WebEnglish Infrastructure Monitoring on AWS Linux 2


## 1. Objectives

* Monitor EC2 performance (CPU, RAM, Disk).
* Collect Spring Boot and MySQL logs.
* Send alerts via SNS.
* Display visual dashboards in CloudWatch.

**Tools used:** AWS CloudWatch, CloudWatch Agent, Amazon SNS.

---

## 2. Prerequisites

### EC2 Instance

- Amazon Linux 2
- Installed: Docker, Docker Compose, Java, MySQL (container or local)

### Required IAM Role Permissions

Attach an IAM role to the EC2 instance with:

- `CloudWatchAgentServerPolicy`
- `AmazonSSMManagedInstanceCore`
- Write permissions for logs and metrics in CloudWatch

Check role attached:
```bash
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
````

### Install AWS CLI (if missing)

```bash
sudo yum install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

Configure CLI (replace with your own credentials/region if needed):

```bash
aws configure
```

---

## 3. Install & Configure CloudWatch Agent

### Step 1: Install Agent

```bash
cd /tmp
curl -O https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
sudo rpm -U ./amazon-cloudwatch-agent.rpm
```
![S3](/images/connect/1.jpg)

### Step 2: Create Configuration File

```bash
sudo nano /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
```

Paste:

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

### Step 3: Start Agent

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
  -s
```
![S3](/images/4.s3/3.jpg)

### Step 4: Verify Agent

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
```

---

## 4. Create Alerts with SNS

### Step 1: Create Topic & Subscribe Email

```bash
aws sns create-topic \
  --name WebEnglishAlerts \
  --region ap-northeast-1
```

Copy `TopicArn` from output.

```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:ap-northeast-1:ACCOUNT_ID:WebEnglishAlerts \
  --protocol email \
  --notification-endpoint YOUR_EMAIL@example.com \
  --region ap-northeast-1
```
![S3](/images/4.s3/5.jpg)

📧 **Confirm the subscription in your email** before continuing.
![S3](/images/4.s3/6.jpg)

---

### Step 2: Create CPU Alarm

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

You can also push fake metrics to CloudWatch for testing:

```bash
aws cloudwatch put-metric-data \
  --metric-name cpu_usage_active \
  --namespace WebEnglishMetrics \
  --value 90
```