---
title : "Deploying an AWS CDK Project Connecting ECS, EC2, IAM"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 2.1.3 </b> "
---

# 🚀 Guide to Deploying an Application on AWS Using AWS CDK

This document provides instructions on how to set up and deploy a Spring Boot application using AWS CDK with the following related services: ECS, EC2 (VPC), IAM, and CloudWatch Logs. The project uses AWS Fargate to run containers without managing servers.

---

## ✅ 1. Installing AWS CDK Environment on Ubuntu

### 🔹 Remove old Node.js/NPM (if any)

```bash
sudo yum remove -y nodejs npm
sudo yum autoremove -y
````

### 🔹 Install NVM and Node.js (Stable version)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
export NVM_DIR="$HOME/.nvm"
source "$NVM_DIR/nvm.sh"
nvm install 22.9.0
nvm use 22.9.0
```

![VPC](/images/2.prerequisite/12-44.jpg)

### 🔹 Install AWS CDK

```bash
npm install -g aws-cdk
```

Check version:

```bash
cdk --version
```

![VPC](/images/2.prerequisite/12-45.jpg)

---

## ✅ 2. Initialize a CDK TypeScript Project

```bash
mkdir test-cdk && cd test-cdk
cdk init app --language typescript
```

![VPC](/images/2.prerequisite/12-46.jpg)

### 🔹 Install additional required AWS libraries

```bash
npm install @aws-cdk/aws-ecs @aws-cdk/aws-ec2 @aws-cdk/aws-ecs-patterns @aws-cdk/aws-rds
```

![VPC](/images/2.prerequisite/12-47.jpg)

---

## ✅ 3. Create CDK Stack (`lib/webenglish-cdk-stack.ts`)

Create the file `lib/webenglish-cdk-stack.ts` with the following content:

```ts
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as ecs from 'aws-cdk-lib/aws-ecs';
import * as iam from 'aws-cdk-lib/aws-iam';
import * as logs from 'aws-cdk-lib/aws-logs';
import { Construct } from 'constructs';

export class TestCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // VPC
    const vpc = new ec2.Vpc(this, 'MyVpc', {
      maxAzs: 2,
      subnetConfiguration: [
        { subnetType: ec2.SubnetType.PUBLIC, name: 'Public' },
        { subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS, name: 'Private' },
      ],
    });

    // ECS Cluster
    const cluster = new ecs.Cluster(this, 'MyCluster', { vpc });

    // IAM Execution Role (existing role)
    const executionRole = iam.Role.fromRoleArn(
      this,
      'EcsExecutionRole',
      'arn:aws:iam::466322313916:role/WebenglishEcsExecutionRole'
    );

    // Log Group
    const logGroup = new logs.LogGroup(this, 'WebenglishLogGroup', {
      retention: logs.RetentionDays.ONE_WEEK,
    });

    // Fargate Task Definition
    const taskDefinition = new ecs.FargateTaskDefinition(this, 'TaskDef', {
      memoryLimitMiB: 2048,
      cpu: 1024,
      executionRole,
    });

    // MySQL Container
    const mysqlContainer = taskDefinition.addContainer('MySQLContainer', {
      image: ecs.ContainerImage.fromRegistry(
        '466322313916.dkr.ecr.ap-northeast-1.amazonaws.com/webenglish:mysql-8.0'
      ),
      environment: {
        MYSQL_ROOT_PASSWORD: '123456',
        MYSQL_DATABASE: 'webenglish',
      },
      logging: ecs.LogDriver.awsLogs({
        logGroup,
        streamPrefix: 'mysql',
      }),
      portMappings: [{ containerPort: 3306 }],
      essential: true,
      healthCheck: {
        command: ['CMD', 'mysqladmin', 'ping', '-h', 'localhost'],
        interval: cdk.Duration.seconds(10),
        timeout: cdk.Duration.seconds(5),
        retries: 5,
        startPeriod: cdk.Duration.seconds(20),
      },
    });

    // App Container (Spring Boot)
    const appContainer = taskDefinition.addContainer('WebenglishApp', {
      image: ecs.ContainerImage.fromRegistry(
        '466322313916.dkr.ecr.ap-northeast-1.amazonaws.com/webenglish:webenglish-app'
      ),
      logging: ecs.LogDriver.awsLogs({
        logGroup,
        streamPrefix: 'webenglish-app',
      }),
      portMappings: [{ containerPort: 8080 }],
      environment: {
        SPRING_DATASOURCE_URL: 'jdbc:mysql://localhost:3306/webenglish',
        SPRING_DATASOURCE_USERNAME: 'root',
        SPRING_DATASOURCE_PASSWORD: '123456',
      },
    });

    // Container Dependency: app waits for MySQL to be healthy
    appContainer.addContainerDependencies({
      container: mysqlContainer,
      condition: ecs.ContainerDependencyCondition.HEALTHY,
    });

    // Fargate Service
    new ecs.FargateService(this, 'WebenglishService', {
      cluster,
      taskDefinition,
      desiredCount: 1,
      assignPublicIp: true,
    });
  }
}
```

---

## ✅ 4. Configure AWS CLI

### 🔹 Log in to AWS

```bash
aws configure
```

Enter:

* `AWS Access Key ID`
* `AWS Secret Access Key`
* `Region`: `ap-northeast-1`

---

## ✅ 5. Create IAM Role for ECS Task

### On AWS Console:

1. Go to **IAM > Roles**
2. Create role `WebenglishEcsExecutionRole`
3. Attach the following policy:

   * `AmazonECSTaskExecutionRolePolicy`
4. Save the ARN of the role:
   Example:
   `arn:aws:iam::466322313916:role/WebenglishEcsExecutionRole`

---

![VPC](/images/2.prerequisite/12-49.jpg)

## ✅ 6. Bootstrap CDK & Deploy

```bash
cdk bootstrap aws://466322313916/ap-northeast-1
cdk synth
cdk deploy
```

![VPC](/images/2.prerequisite/12-53.jpg)
![VPC](/images/2.prerequisite/12-52.jpg)
![VPC](/images/2.prerequisite/12-54.jpg)
![VPC](/images/2.prerequisite/12-55.jpg)
![VPC](/images/2.prerequisite/12-56.jpg)
![VPC](/images/2.prerequisite/12-57.jpg)
![VPC](/images/2.prerequisite/12-570.jpg)
![VPC](/images/2.prerequisite/12-571.jpg)

---

## 📌 Additional Notes

* You must create the ECR repository and push the image `webenglish:webenglish-app` beforehand.
* If using a real RDS database, you must create an RDS instance and update the `SPRING_DATASOURCE_URL`.

---

## 🧠 Future Improvements

* Integrate a Load Balancer (ALB) using `ApplicationLoadBalancedFargateService`
* Connect RDS via Security Group and Secrets Manager
* Integrate CI/CD with GitHub Actions or AWS CodePipeline

---

## 🧾 References

* [https://docs.aws.amazon.com/cdk/](https://docs.aws.amazon.com/cdk/)
* [https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)

```

---

