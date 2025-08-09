---
title : " Hướng Dẫn Tạo EC2 Instance Default Trên AWS"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 2.1.1 </b> "
---

---

---

# 🖥️ Hướng Dẫn Tạo EC2 Instance Default Trên AWS

## 📌 Mục Tiêu
Tạo một EC2 instance sử dụng cấu hình mặc định để triển khai ứng dụng hoặc kiểm thử môi trường.

## 🧰 Yêu Cầu Trước Khi Bắt Đầu
- Tài khoản AWS hợp lệ
- Đăng nhập AWS Console hoặc cài sẵn AWS CLI
- Tạo key pair để SSH (nếu cần)

## 🛠️ Các Bước Thực Hiện

### 1. Đăng Nhập AWS Console
Truy cập: [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/)

### 2. Tạo EC2 Instance

#### Cách 1: Qua AWS Console
1. Vào **EC2 Dashboard**
2. Nhấn **Launch Instance**
3. Nhập tên: `MyEC2Default`
4. **Chọn AMI**: `Amazon Linux 2023` (hoặc Amazon Linux 2)
5. **Loại Instance**: `t2.micro` (Free tier)
6. **Key pair**: Chọn hoặc tạo mới
7. **Network Settings**:
   - Allow SSH (port 22)
   - Allow HTTP (port 80) nếu cần
8. **Ổ đĩa**: Mặc định 8 GB (gp2)
9. Nhấn **Launch Instance**
![VPC](/images/2.prerequisite/12-1.jpg)
![VPC](/images/2.prerequisite/12-2.jpg)
![VPC](/images/2.prerequisite/12-3.jpg)
![VPC](/images/2.prerequisite/12-4.jpg)
![VPC](/images/2.prerequisite/12-5.jpg)
![VPC](/images/2.prerequisite/12-6.jpg)
![VPC](/images/2.prerequisite/12-7.jpg)
![VPC](/images/2.prerequisite/12-8.jpg)
#### Cách 2: Dùng AWS CLI
```bash
aws ec2 run-instances   --image-id ami-0c02fb55956c7d316 \ # Amazon Linux 2 (us-east-1)
  --instance-type t2.micro   --key-name my-key   --security-groups default   --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyEC2Default}]'
```

> 📝 Ghi chú: Thay `ami-xxxx` và `my-key` bằng ID và key pair tương ứng trong region của bạn.

### 3. Kết Nối EC2 Bằng SSH
```bash
ssh -i my-key.pem ec2-user@<public-ip>
```

## ✅ Kết Quả
- EC2 instance được tạo và chạy trong vài phút.
- Có thể SSH để cài đặt thêm hoặc triển khai ứng dụng.
![VPC](/images/2.prerequisite/12-9.jpg)

## 🧹 Mẹo Quản Lý
- Tắt hoặc terminate instance sau khi dùng để tránh mất phí.
- Gắn Elastic IP nếu muốn giữ IP cố định.

## 📚 Tài Liệu Tham Khảo
- [EC2 User Guide](https://docs.aws.amazon.com/ec2/index.html)
- [Amazon Linux AMI](https://aws.amazon.com/amazon-linux-ami/)
