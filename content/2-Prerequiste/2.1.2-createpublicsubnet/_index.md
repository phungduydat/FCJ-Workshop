---
title : "Deploying a Docker Image to AWS ECR on Linux"
date :  "`r Sys.Date()`" 
weight : 3
chapter : false
pre : " <b> 2.1.2 </b> "
---

## 🎯 Objective

This guide explains how to install AWS CLI, configure your AWS account, create a repository on Amazon ECR, and push Docker images from a local machine (including MySQL images) to ECR.

---

## 🧰 1. Install AWS CLI on Linux

Run the following commands to install AWS CLI v2:

```bash
# Download AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Unzip
unzip awscliv2.zip

# Install
sudo ./aws/install

# Verify version
aws --version
````

![VPC](/images/2.prerequisite/12-30.jpg)
![VPC](/images/2.prerequisite/12-31.jpg)

---

## ⚙️ 2. Configure AWS CLI

After installation, run the following to configure your AWS credentials:

```bash
aws configure
```

![VPC](/images/2.prerequisite/12-33.jpg)
![VPC](/images/2.prerequisite/12-34.jpg)

Enter the following information:

* `AWS Access Key ID`: from your IAM user
* `AWS Secret Access Key`: from your IAM user
* `Default region name`: `ap-northeast-1` (or your preferred region)
* `Default output format`: `json`

![VPC](/images/2.prerequisite/12-35.jpg)

---

## 📦 3. Create ECR Repository on AWS Console

1. Go to: [https://console.aws.amazon.com/ecr](https://console.aws.amazon.com/ecr)
2. Select **Repositories** → **Create repository**
3. Enter the repository name: `webenglish`
4. Select **Private**, keep default settings
5. Click **Create repository**

![VPC](/images/2.prerequisite/12-36.jpg)
![VPC](/images/2.prerequisite/12-37.jpg)

You will receive a URI like this:

```
466322313916.dkr.ecr.ap-northeast-1.amazonaws.com/webenglish
```

---

## 🏗 4. Build the Application Docker Image

Navigate to the folder containing the `Dockerfile` and build the image:

```bash
docker build -t webenglish-app .
```

---

## 🔐 5. Log In to Amazon ECR

Before pushing, you must log in to ECR:

```bash
aws ecr get-login-password --region ap-northeast-1 | \
docker login --username AWS \
--password-stdin 466322313916.dkr.ecr.ap-northeast-1.amazonaws.com
```

![VPC](/images/2.prerequisite/12-38.jpg)

---

## 🏷 6. Tag the Docker Image with ECR URI

```bash
docker tag webenglish-app:latest \
466322313916.dkr.ecr.ap-northeast-1.amazonaws.com/webenglish:webenglish-app
```

![VPC](/images/2.prerequisite/12-39.jpg)

---

## 🚀 7. Push the Docker Image to ECR

```bash
docker push \
466322313916.dkr.ecr.ap-northeast-1.amazonaws.com/webenglish:webenglish-app
```

![VPC](/images/2.prerequisite/12-40.jpg)

---

## 🐬 8. Push MySQL 8.0 Image to ECR (Optional)

If you already pulled/built the MySQL `8.0` image and its ID is `7d4e34ccfad4`, you can tag and push it as follows:

### ✅ Tag the MySQL image

```bash
docker tag 7d4e34ccfad4 \
466322313916.dkr.ecr.ap-northeast-1.amazonaws.com/webenglish:mysql-8.0
```

![VPC](/images/2.prerequisite/12-41.jpg)

📌 **Note**: You can replace `mysql-8.0` with `latest` if preferred.

### ✅ Push the MySQL image to ECR

```bash
docker push \
466322313916.dkr.ecr.ap-northeast-1.amazonaws.com/webenglish:mysql-8.0
```

![VPC](/images/2.prerequisite/12-42.jpg)

⏳ MySQL image size is around 772MB — pushing may take a few minutes.

---

## 📋 9. Verify in AWS Console

After pushing:

* Go back to AWS Console → ECR
* Open the `webenglish` repository
* Verify that images with tags like `webenglish-app`, `mysql-8.0`, etc., are present

![VPC](/images/2.prerequisite/12-43.jpg)

---

## 📝 10. Additional Notes

* Docker must be installed and running:

  ```bash
  sudo systemctl start docker
  ```
* Your IAM account must have the following permission:

  * `AmazonEC2ContainerRegistryFullAccess`
* You can use ECR images to deploy containers on:

  * EC2
  * ECS
  * EKS

---

## 📚 11. References

* [AWS CLI Install Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)
* [Amazon ECR Documentation](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)
* [Docker CLI Docs](https://docs.docker.com/engine/reference/commandline/cli/)

```

---
