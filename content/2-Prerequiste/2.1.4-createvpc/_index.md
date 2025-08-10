---
title : "Preparing the Environment to Create a Docker Image for a Spring Boot Project"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 2.1.2 </b> "
---

---

---

## 🔧 1. Install Java (OpenJDK 21)

To run the Spring Boot application and build the `.jar` file, you need Java JDK 21.

```bash
# Update package list and upgrade the system
sudo yum update -y

# Install Java OpenJDK 21
sudo amazon-linux-extras enable corretto8
sudo yum install -y java-21-amazon-corretto-devel

# Verify installed version
java -version
````

![VPC](/images/2.prerequisite/12-30.jpg)

---

## 🔧 2. Install Maven

### 📥 Download Maven

Download the latest Maven binary (version **3.9.11** as of July 2025):

```bash
sudo yum install java-11-amazon-corretto-devel
wget https://archive.apache.org/dist/maven/maven-3/3.9.1/binaries/apache-maven-3.9.1-bin.tar.gz
```

![VPC](/images/2.prerequisite/12-58.jpg)

### 📦 Extract Maven

```bash
tar xzvf apache-maven-3.9.1-bin.tar.gz  
sudo mv apache-maven-3.9.1 /opt/
```

![VPC](/images/2.prerequisite/12-59.jpg)
![VPC](/images/2.prerequisite/12-60.jpg)

### ⚙️ Configure Environment Variables

```bash
echo 'export M2_HOME=/opt/apache-maven-3.9.1' >> ~/.bashrc
echo 'export PATH=$PATH:$M2_HOME/bin' >> ~/.bashrc
source ~/.bashrc
```

### ✅ Verify Maven

```bash
mvn -version
```

![VPC](/images/2.prerequisite/12-61.jpg)

---

## 🐳 3. Install Docker

### 🧹 Remove Old Docker Versions (if any)

```bash
sudo dnf remove -y docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
```

![VPC](/images/2.prerequisite/12-14.jpg)

### 📦 Install Dependencies

```bash
sudo dnf install -y dnf-plugins-core device-mapper-persistent-data lvm2
```

### 🗂️ Add Docker CE Repository

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
sudo sed -i 's/$releasever/41/g' /etc/yum.repos.d/docker-ce.repo
```

### 🐳 Install Docker 28.1.1

```bash
sudo dnf install -y docker-ce-28.1.1 docker-ce-cli-28.1.1 containerd.io docker-buildx-plugin docker-compose-plugin
```

If version 28.1.1 is not listed:

```bash
dnf list --showduplicates docker-ce
```

### 🚀 Start Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

---

## 🧱 4. Install Docker Compose

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

![VPC](/images/2.prerequisite/12-17.jpg)

---

## 📆 5. Install Git & Clone the Project

```bash
sudo yum install -y git
git clone https://github.com/phungduydat/webenglish.git
cd webenglish
```

![VPC](/images/2.prerequisite/12-62.jpg)

---

## 🚰 6. Build Spring Boot (Skip Tests)

```bash
mvn clean package -DskipTests
```

![VPC](/images/2.prerequisite/12-25.jpg)

The jar file will be located in the `target/` directory.

---

## 📄 7. Create Dockerfile

```dockerfile
FROM openjdk:21-jdk
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

![VPC](/images/2.prerequisite/12-26.jpg)

---

## 🧩 8. Create docker-compose.yml File

```yaml
services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: webenglish
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  app:
    build: .
    container_name: webenglish-app
    ports:
      - "8080:8090"
    depends_on:
      - mysql
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/webenglish
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: 123456
    volumes:
      - ./src/main/resources:/app/resources

volumes:
  mysql_data:
```

![VPC](/images/2.prerequisite/12-27.jpg)

---

## ⚙️ 9. Configure `application.properties`

```properties
spring.datasource.url=${SPRING_DATASOURCE_URL}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD}
```

![VPC](/images/2.prerequisite/12-28.jpg)

---

## 🧪 10. Build & Run Docker Compose

```bash
docker compose up --build
```

![VPC](/images/2.prerequisite/12-29.jpg)

Check:

```bash
docker ps
docker compose logs
```

---

**Author:** Phùng Duy Đạt
**GitHub:** [https://github.com/phungduydat/webenglish](https://github.com/phungduydat/webenglish)

```

---

