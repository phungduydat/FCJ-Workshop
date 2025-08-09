---
title : " Guide to Launching a Default EC2 Instance on AWS"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 2.1.1 </b> "
---

---

---

# 🖥️ Guide to Launching a Default EC2 Instance on AWS

## 📌 Objective
Launch an EC2 instance using default settings for application deployment or environment testing.

## 🧰 Prerequisites
- A valid AWS account  
- Signed in to AWS Console or AWS CLI installed  
- A key pair created for SSH access (if needed)

## 🛠️ Step-by-Step Instructions

### 1. Sign in to AWS Console
Visit: [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/)

### 2. Launch an EC2 Instance

#### Method 1: Using AWS Console
1. Go to **EC2 Dashboard**
2. Click **Launch Instance**
3. Enter name: `MyEC2Default`
4. **Choose AMI**: `Amazon Linux 2023` (or Amazon Linux 2)
5. **Instance Type**: `t2.micro` (Free tier eligible)
6. **Key pair**: Choose existing or create new
7. **Network Settings**:
   - Allow SSH (port 22)
   - Allow HTTP (port 80) if needed
8. **Storage**: Default 8 GB (gp2)
9. Click **Launch Instance**
![VPC](/images/2.prerequisite/12-1.jpg)
![VPC](/images/2.prerequisite/12-2.jpg)
![VPC](/images/2.prerequisite/12-3.jpg)
![VPC](/images/2.prerequisite/12-4.jpg)
![VPC](/images/2.prerequisite/12-5.jpg)
![VPC](/images/2.prerequisite/12-6.jpg)
![VPC](/images/2.prerequisite/12-7.jpg)
![VPC](/images/2.prerequisite/12-8.jpg)

#### Method 2: Using AWS CLI
```bash
aws ec2 run-instances --image-id ami-0c02fb55956c7d316 \ # Amazon Linux 2 (us-east-1)
  --instance-type t2.micro \
  --key-name my-key \
  --security-groups default \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyEC2Default}]'
