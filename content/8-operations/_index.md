---
title: "Operational Procedures"
date: "`r Sys.Date()`"
weight: 7
chapter: false
pre: "<b>7. </b>"
---

# ✅ Operational Procedures & Optimization

## 🔹 1. Creating a Detailed Runbook

A **Runbook** is a step-by-step guide for handling common incident scenarios, including:

| Incident Scenario         | Symptoms                                    | Common Cause                              | Detailed Resolution Steps                                                                   | Responsible Role       |
|---------------------------|---------------------------------------------|--------------------------------------------|----------------------------------------------------------------------------------------------|------------------------|
| CPU at 100%               | CloudWatch alarm “HighCPUUsage”            | Sudden traffic spike                       | 1. Check application logs  <br> 2. Verify running ECS containers <br> 3. Scale ECS service by adding 1 task | DevOps Engineer        |
| Application not responding| Client timeout, “TargetResponseTimeHigh” alarm | ECS service overload or application bug    | 1. Check application logs <br> 2. Manually scale the service if needed <br> 3. Configure auto-scaling alarm | DevOps Engineer        |

> 📁 **Suggested storage location**: `/runbook/webenglish/ecs-cpu-spike.md`

---

## 🔹 2. Integrating AWS DevOps Guru

### 🧠 Introduction

**AWS DevOps Guru** is a service that uses AI to detect anomalies and recommend remediation actions for your system.

### 🎯 Key Benefits

- Automatically detects incidents without manual rule configuration
- Analyzes logs, metrics, and traces
- Suggests root causes and remediation actions
- Integrates with SNS for alert notifications
- Provides visual representation via Anomaly Maps

---

## 🔹 3. Enabling DevOps Guru for the System

### 🔧 Deployment Steps

1. Navigate to **AWS Console > DevOps Guru**
2. Click **Enable DevOps Guru**
3. Select the monitoring scope:
   - **Analyze all AWS resources** *(monitor all resources)*
   - Or select a specific **CloudFormation Stack**
4. Configure **SNS Topic** for alerts (create one if not available)
5. Click **Enable**

> ⏳ DevOps Guru will start collecting data and provide insights within a few minutes to a few hours.  
![FWD](/images/8/1.png)  
![FWD](/images/8/2.png)  
![FWD](/images/8/3.png)  
![FWD](/images/8/4.png)

---

## 🔹 4. Monitoring & Receiving Alerts

### 🧾 Insight details include:

- Affected resource name (ECS, RDS, Lambda…)
- Time of occurrence
- Abnormal metrics
- Suggested root cause
- Recommended remediation actions

### 📩 Setting up SNS for alerts

- Create an SNS Topic: `devops-guru-alerts`
- Add subscribers: email, Lambda, webhook…

---

## 🔹 5. Combining Services for Optimization

| Service       | Purpose                                             |
|---------------|-----------------------------------------------------|
| DevOps Guru   | Automatic incident detection and analysis           |
| SNS           | Send alerts to email, Slack, Lambda...              |
| CloudWatch    | Collect metrics, logs, and create monitoring charts |
| Notebook      | Custom queries and advanced analysis                |

---

## 📌 Advanced Recommendations

- Deploy DevOps Guru to reduce incident investigation time
- Combine Amazon QuickSight + SNS to send scheduled reports
- Automate incident remediation using AWS Lambda or Systems Manager

---

## 📚 References

- [🔗 AWS CloudWatch Log Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html)
- [🔗 Auto Scaling ECS](https://docs.aws.amazon.com/autoscaling/application/userguide/application-auto-scaling.html)
- [🔗 AWS DevOps Guru](https://docs.aws.amazon.com/devops-guru/latest/userguide/what-is-devops-guru.html)
- [🔗 AWS Runbook Template](https://aws.amazon.com/blogs/devops/using-aws-systems-manager-to-create-runbooks/)
