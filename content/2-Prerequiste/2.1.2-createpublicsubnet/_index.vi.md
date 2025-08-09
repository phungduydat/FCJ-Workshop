---
title : "Deploying a Docker Image to AWS ECR on Linux "
date :  "`r Sys.Date()`" 
weight : 3
chapter : false
pre : " <b> 2.1.2 </b> "
---


## 🎯 Mục Tiêu

Hướng dẫn cài đặt AWS CLI, cấu hình tài khoản AWS, tạo repository trên Amazon ECR, và đẩy Docker image từ máy local (bao gồm cả image MySQL) lên ECR.

---

## 🧰 1. Cài Đặt AWS CLI Trên Linux

Chạy các lệnh sau để cài đặt AWS CLI v2:

```bash
# Tải AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Giải nén
unzip awscliv2.zip

# Cài đặt
sudo ./aws/install

# Kiểm tra phiên bản
aws --version
```
![VPC](/images/2.prerequisite/12-30.jpg)
![VPC](/images/2.prerequisite/12-31.jpg)

---

## ⚙️ 2. Cấu Hình AWS CLI

Sau khi cài đặt, chạy lệnh sau để cấu hình thông tin tài khoản:

```bash
aws configure
```
![VPC](/images/2.prerequisite/12-33.jpg)
![VPC](/images/2.prerequisite/12-34.jpg)
Nhập các thông tin:

- `AWS Access Key ID`: từ IAM user  
- `AWS Secret Access Key`: từ IAM user  
- `Default region name`: `ap-northeast-1` (hoặc vùng bạn sử dụng)  
- `Default output format`: `json`

---
![VPC](/images/2.prerequisite/12-35.jpg)

## 📦 3. Tạo ECR Repository Trên AWS Console

1. Truy cập: [https://console.aws.amazon.com/ecr](https://console.aws.amazon.com/ecr)  
2. Chọn **Repositories** → **Create repository**  
3. Nhập tên repository: `webenglish`  
4. Chọn **Private**, giữ thiết lập mặc định  
5. Nhấn **Create repository**  

![VPC](/images/2.prerequisite/12-36.jpg)
![VPC](/images/2.prerequisite/12-37.jpg)



Bạn sẽ nhận được URI như sau:

```
466322313916.dkr.ecr.ap-northeast-1.amazonaws.com/webenglish
```

---

## 🏗 4. Xây Dựng Docker Image Ứng Dụng

Di chuyển đến thư mục có `Dockerfile` và build image:

```bash
docker build -t webenglish-app .
```

---

## 🔐 5. Đăng Nhập Vào Amazon ECR

Trước khi push, bạn cần đăng nhập vào ECR:

```bash
aws ecr get-login-password --region ap-northeast-1 | \
docker login --username AWS \
--password-stdin 466322313916.dkr.ecr.ap-northeast-1.amazonaws.com
```
![VPC](/images/2.prerequisite/12-38.jpg)

---

## 🏷 6. Tag Docker Image Với ECR URI

```bash
docker tag webenglish-app:latest \
466322313916.dkr.ecr.ap-northeast-1.amazonaws.com/webenglish:webenglish-app
```
![VPC](/images/2.prerequisite/12-39.jpg)


---

## 🚀 7. Push Docker Image Lên ECR

```bash
docker push \
466322313916.dkr.ecr.ap-northeast-1.amazonaws.com/webenglish:webenglish-app
```
![VPC](/images/2.prerequisite/12-40.jpg)

---

## 🐬 8. Đẩy Image MySQL 8.0 Lên ECR (Tuỳ chọn)

Nếu bạn đã pull/build image MySQL `8.0` và image ID là `7d4e34ccfad4`, bạn có thể tag và push như sau:

### ✅ Gắn tag MySQL image

```bash
docker tag 7d4e34ccfad4 \
466322313916.dkr.ecr.ap-northeast-1.amazonaws.com/webenglish:mysql-8.0
```
![VPC](/images/2.prerequisite/12-41.jpg)

📌 **Lưu ý**: Bạn có thể thay `mysql-8.0` bằng `latest` nếu muốn.

### ✅ Push MySQL image lên ECR

```bash
docker push \
466322313916.dkr.ecr.ap-northeast-1.amazonaws.com/webenglish:mysql-8.0
```
![VPC](/images/2.prerequisite/12-42.jpg)

⏳ Kích thước image MySQL khoảng 772MB — quá trình push có thể mất vài phút.

---

## 📋 9. Kiểm Tra Trên AWS Console

Sau khi push xong:

- Truy cập lại AWS Console → ECR  
- Vào repository `webenglish`  
- Xác minh đã có các image với tag: `webenglish-app`, `mysql-8.0`, v.v.

![VPC](/images/2.prerequisite/12-43.jpg)

---

## 📝 10. Lưu Ý Bổ Sung

- Docker cần được cài đặt và chạy:
  ```bash
  sudo systemctl start docker
  ```
- Tài khoản IAM phải có quyền:
  - `AmazonEC2ContainerRegistryFullAccess`
- Bạn có thể sử dụng image từ ECR để triển khai container trên:
  - EC2
  - ECS
  - EKS

---

## 📚 11. Tài Liệu Tham Khảo

- [AWS CLI Install Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)  
- [Amazon ECR Documentation](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)  
- [Docker CLI Docs](https://docs.docker.com/engine/reference/commandline/cli/)

---
