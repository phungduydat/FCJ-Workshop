---
title : "Preparation Steps"
date :  "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

{{% notice info %}}
To get started, you need to prepare an AWS environment including an EC2 instance to run Docker, connect it to Amazon ECR, and deploy your application using AWS CDK. Once deployed, you will be able to access the application via your web browser.
{{% /notice %}}

If you’re not familiar with the core components like EC2, ECR, or CDK, you can refer to the following resources:

- [Introduction to Amazon EC2](https://000004.awsstudygroup.com/vi/)
- [Amazon Elastic Container Registry (ECR)](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)
- [AWS Cloud Development Kit (CDK)](https://docs.aws.amazon.com/cdk/latest/guide/home.html)

In this section, we’ll walk through the essential preparation steps to get your containerized application ready for deployment.

### Overview
- [Create an EC2 Instance and Set Up Docker](2.1-create-ec2-docker/)
- [Push a Docker Image to Amazon ECR](2.2-upload-to-ecr/)
- [Deploy the Application Using AWS CDK](2.3-deploy-cdk/)
- [Verify the Deployment in Your Browser](2.4-access-web/)
