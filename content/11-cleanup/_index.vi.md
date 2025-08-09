
---
title: "Clean up"
date: "`r Sys.Date()`"
weight: 8
chapter: false
pre: "<b>8. </b>"
---

Chúng ta sẽ tiến hành **dọn dẹp toàn bộ tài nguyên AWS** đã tạo trong quá trình thực hành. Hãy thực hiện theo các bước sau để tránh phát sinh chi phí.

---

### ✅ Xóa DevOps Guru

1. Truy cập [DevOps Guru Console](https://console.aws.amazon.com/devops-guru/home)
2. Ở menu trái, chọn **Settings**.
3. Tại mục **CloudFormation stacks**, click **Delete** đối với stack DevOps Guru đã kích hoạt.
4. Xác nhận xóa nếu được yêu cầu.
---

### ✅ Xóa CDK Stack bằng `cdk destroy`

1. Mở terminal tại thư mục chứa project CDK.
2. Chạy lệnh sau để xóa toàn bộ stack CDK:
   ```bash
   cdk destroy
   ```
3. Khi được hỏi xác nhận, nhập **y** và nhấn Enter.
4. Chờ quá trình hủy stack hoàn tất.
![Clean](/images/6.clean/1.png)
![Clean](/images/6.clean/2.png)
![Clean](/images/6.clean/3.png)

---

### ✅ Xóa S3 Bucket

1. Truy cập [S3 Console](https://s3.console.aws.amazon.com/s3/home)
2. Click chọn các bucket bạn đã tạo (VD: `cdk-bucket-*`, `log-bucket-*`, …).
3. Thực hiện lần lượt các bước:
   + Click **Empty** → Gõ `permanently delete` → Click **Empty**.
   + Sau đó, click **Delete** → Gõ tên bucket → Click **Delete bucket**.

---

### ✅ Xóa CloudFormation Stack (nếu không dùng CDK)

1. Truy cập [CloudFormation Console](https://console.aws.amazon.com/cloudformation/home)
2. Chọn các stack liên quan đến bài lab (VD: `DevOpsGuruStack`, `CDKStack`, …).
3. Click **Delete** và xác nhận xóa stack.

---

### ✅ Xóa CloudWatch Logs / Alarms / Dashboards

1. Truy cập [CloudWatch Console](https://console.aws.amazon.com/cloudwatch/home)

**Xóa Log Groups:**

- Vào **Logs > Log groups**  
- Chọn các log group (VD: `/aws/lambda/...`, `/aws/events/...`)  
- Click **Actions > Delete log groups**

**Xóa Alarms:**

- Vào **Alarms > All alarms**  
- Chọn tất cả alarms → Click **Actions > Delete**

**Xóa Dashboards:**

- Vào **Dashboards**  
- Chọn dashboard → Click **Actions > Delete**
![Clean](/images/6.clean/4.png)
![Clean](/images/6.clean/5.png)

---

### ✅ Xóa EC2 Instances

1. Truy cập [EC2 Console](https://console.aws.amazon.com/ec2/v2/home)
2. Vào **Instances**
3. Chọn các instance cần xóa → Click **Instance state > Terminate instance**
4. Xác nhận **Terminate**
![Clean](/images/6.clean/6.png)

---

### ✅ Xóa VPC

1. Truy cập [VPC Console](https://console.aws.amazon.com/vpc/home)
2. Vào **Your VPCs**
3. Chọn VPC bạn đã tạo → Click **Actions > Delete VPC**
4. Xác nhận xóa

---

### ✅ Xóa Subnet (nếu không tự động xóa cùng VPC)

1. Trong VPC Console → Vào mục **Subnets**
2. Chọn các subnet (Public, Private) liên quan → Click **Actions > Delete subnet**

---

> ✅ **Lưu ý:** Sau khi hoàn tất dọn dẹp, bạn có thể vào **Billing Dashboard** để kiểm tra chi phí và đảm bảo không có tài nguyên nào còn tồn tại.

---
