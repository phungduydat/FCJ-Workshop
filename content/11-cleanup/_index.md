---
title: "Clean up"
date: "`r Sys.Date()`"
weight: 8
chapter: false
pre: "<b>8. </b>"
---
We will proceed to **clean up all AWS resources** created during the practice session. Please follow the steps below to avoid incurring unnecessary charges.

---

### ✅ Delete DevOps Guru

1. Go to the [DevOps Guru Console](https://console.aws.amazon.com/devops-guru/home)
2. On the left menu, choose **Settings**.
3. Under **CloudFormation stacks**, click **Delete** for the DevOps Guru stack you activated.
4. Confirm deletion if prompted.

---

### ✅ Destroy CDK Stack using `cdk destroy`

1. Open the terminal in the directory containing your CDK project.
2. Run the following command to destroy the CDK stack:
   ```bash
   cdk destroy
   ```
3. When prompted, type **y** and press Enter.
4. Wait for the stack destruction to complete.
![Clean](/images/6.clean/1.png)
![Clean](/images/6.clean/2.png)
![Clean](/images/6.clean/3.png)

---

### ✅ Delete S3 Bucket

1. Go to the [S3 Console](https://s3.console.aws.amazon.com/s3/home)
2. Select the buckets you created (e.g., `cdk-bucket-*`, `log-bucket-*`, etc.).
3. Perform the following steps:
   + Click **Empty** → Type `permanently delete` → Click **Empty**.
   + Then, click **Delete** → Enter the bucket name → Click **Delete bucket**.

---

### ✅ Delete CloudFormation Stack (if not using CDK)

1. Go to the [CloudFormation Console](https://console.aws.amazon.com/cloudformation/home)
2. Select the stacks related to the lab (e.g., `DevOpsGuruStack`, `CDKStack`, etc.).
3. Click **Delete** and confirm deletion.

---

### ✅ Delete CloudWatch Logs / Alarms / Dashboards

1. Go to the [CloudWatch Console](https://console.aws.amazon.com/cloudwatch/home)

**Delete Log Groups:**

- Go to **Logs > Log groups**  
- Select the log groups (e.g., `/aws/lambda/...`, `/aws/events/...`)  
- Click **Actions > Delete log groups**

**Delete Alarms:**

- Go to **Alarms > All alarms**  
- Select all alarms → Click **Actions > Delete**

**Delete Dashboards:**

- Go to **Dashboards**  
- Select dashboard → Click **Actions > Delete**
![Clean](/images/6.clean/4.png)
![Clean](/images/6.clean/5.png)

---

### ✅ Terminate EC2 Instances

1. Go to the [EC2 Console](https://console.aws.amazon.com/ec2/v2/home)
2. Go to **Instances**
3. Select the instances to delete → Click **Instance state > Terminate instance**
4. Confirm **Terminate**
![Clean](/images/6.clean/6.png)

---

### ✅ Delete VPC

1. Go to the [VPC Console](https://console.aws.amazon.com/vpc/home)
2. Go to **Your VPCs**
3. Select the VPC you created → Click **Actions > Delete VPC**
4. Confirm deletion

---

### ✅ Delete Subnets (if not automatically removed with VPC)

1. In the VPC Console → Go to **Subnets**
2. Select the related subnets (Public, Private) → Click **Actions > Delete subnet**

---

> ✅ **Note:** After cleaning up, go to the **Billing Dashboard** to check for any remaining resources and ensure no extra costs are incurred.

---
