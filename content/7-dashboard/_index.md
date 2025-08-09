---
title : "Dashboard Development"
date :  "`r Sys.Date()`" 
weight : 5 
chapter : false
pre : " <b> 5. </b> "
---

# 📊 Implementing Dashboards and Visual Reports for the WebEnglish Project

This document provides a professional and detailed process for setting up dashboards and system data monitoring reports for the WebEnglish project. Monitoring is implemented using Amazon CloudWatch and Amazon QuickSight to deliver real-time visibility as well as strategic analytics, enabling the DevOps team to respond quickly to product issues.

---

## 🎯 Objectives

- Display the performance of EC2, containers, Spring Boot applications, and MySQL.
- Trigger alerts when anomalies are detected based on behavioral patterns.
- Provide regular visual reports for DevOps and senior management.

---

## 🧭 Step 1: Access the CloudWatch Dashboard
- Go to AWS Console → Search for “CloudWatch”.
- Go to **Dashboards** → Select or create a new dashboard.  
![FWD](/images/9/2.jpg)  
![FWD](/images/9/3.jpg)

## 📊 Step 2: Add Metric Charts
- Click **“Add widget”** → Choose **Line** → Click **Next**.
- In the “Browse” section, select:
  - Region: Tokyo (or where your instance is running)
  - Namespace: `WebEnglishMetrics` (as configured)
  - Metric: `cpu_usage_active`  
![FWD](/images/9/4.jpg)  
![FWD](/images/9/5.jpg)  
![FWD](/images/9/6.jpg)

## 📈 Step 3: Add Anomaly Detection Band
- In **Graphed metrics**, click **“Add math”** → choose `ANOMALY_DETECTION_BAND(m1, 2)`
  - `m1` is the original metric ID.
  - `2` is the sensitivity level.  
![FWD](/images/9/7.jpg)  
![FWD](/images/9/8.jpg)

## 🚨 Step 4: Create an Alarm from the Metric
- Go to the **Actions** tab → Click **“Create Alarm”**
- Select the metric `cpu_usage_active`
- Choose statistic type: `Average`
- Period: `5 minutes`
- **Conditions**:
  - Threshold type: `Anomaly detection`
  - Anomaly detection model: select the line `ANOMALY_DETECTION_BAND(m1, 2)`  
![FWD](/images/9/9.jpg)  
![FWD](/images/9/10.jpg)

## ⚙️ Step 5: Additional Configuration

| Option | Recommended setting |
|--------|----------------------|
| **Datapoints to alarm** | `3 out of 5` |
| **Missing data treatment** | `Treat missing data as missing` |
| **Actions suppression** | Skip if no other rule is in use |

## 🏁 Step 6: Name & Create Alarm
- Name the alarm, for example: `HighCPUWithAnomalyDetection`
- Select an SNS topic if you want to send emails
- Click **Create Alarm**
---

## 🧾 Operational Tips

- ⏱️ Limit log retention: `7–14 days` to reduce costs.
- 🔔 Configure alerts via **SNS Topic** to email or Lambda handler.
- 🧪 Combine with **AWS DevOps Guru** for automatic root cause detection.

---

## 📚 References

- [CloudWatch Dashboards](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html)
- [Amazon QuickSight Getting Started](https://docs.aws.amazon.com/quicksight/latest/user/welcome.html)
- [Anomaly Detection](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Anomaly_Detection.html)
