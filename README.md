# php_cicd_pipeline-project

## Replace These Before Use:
## <SONARQUBE_EC2_PRIVATE_IP> with your SonarQube EC2 private IP or hostname

## <NEXUS_EC2_PRIVATE_IP> with your Nexus EC2 private IP

## Replace your_sonar_token with your actual SonarQube token

## Update Docker credentials ID and registry if using Nexus Docker registry

## Why use Private IP over Public IP in Jenkins pipelines on AWS EC2?
## Use Private IP when:
## All your servers (Jenkins, Nexus, SonarQube) are in the same VPC / subnet.

## You want faster, cheaper, and more secure communication between instances.

## You are not exposing services like Nexus and SonarQube to the internet.

## When Public IP is OK:
## Services are in different VPCs or regions without VPC peering.

## You haven't configured private subnets or internal routing yet.

## You're still testing and haven't set up VPC networking...


# Full Setup Guide: Jenkins, SonarQube, and Nexus on AWS EC2
 This guide will help you install **Jenkins**, **SonarQube**, and **Nexus Repository
 Manager** on separate EC2 instances and link them together for a full PHP CI/CD
 pipeline.--
## 1. EC2 Instances Setup
 Launch **3 EC2 Instances** (e.g., `t3.medium` or `t3.small`) with Ubuntu 22.04 and the
 following names:- `jenkins-server`- `sonarqube-server`- `nexus-server`
 **Open these ports** in the security groups:
 | Instance        | Open Ports                |
 |-----------------|---------------------------|
 | Jenkins         | 22, 8080                  |
 | SonarQube       | 22, 9000                  |
 | Nexus           | 22, 8081 (UI), 8082 (Docker) |--
## 2. Install Docker and Docker Compose (on All 3 Instances)
 ```bash
 sudo apt update && sudo apt install -y docker.io docker-compose
 sudo usermod -aG docker ubuntu
 newgrp docker
 ```--
## 3. SonarQube Setup (on `sonarqube-server`)
 ### A. Run SonarQube Container
 ```bash
 docker run -d --name sonarqube \
  -p 9000:9000 \
  -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
  sonarqube:lts
 ```
 Wait ~2 minutes and visit: `http://<sonarqube-ec2-ip>:9000`  
Default credentials: `admin / admin`
 ### B. Generate Token for Jenkins
- Go to **My Account > Security**- Create token: `sonar-auth`--
## 4. Nexus Repository Setup (on `nexus-server`)
 ### A. Run Nexus Container
 ```bash
 docker run -d --name nexus \
  -p 8081:8081 -p 8082:8082 \
  sonatype/nexus3
 ```
 Visit: `http://<nexus-ec2-ip>:8081`  
Get password from:
 ```bash
 docker exec -it nexus cat /nexus-data/admin.password
 ```
 ### B. Create Docker Repository- Login to Nexus UI- Go to **Repositories > Create Repository**- Select **docker (hosted)**- Set name: `docker-local`- HTTP port: `8082`- Deployment policy: `Allow Redeploy`--
## 5. Jenkins Setup (on `jenkins-server`)
 ### A. Install Jenkins
 ```bash
 sudo apt update
 sudo apt install -y openjdk-17-jdk
 wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee
 /usr/share/keyrings/jenkins-keyring.asc
 echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list
 sudo apt update
 sudo apt install -y jenkins
 sudo systemctl enable jenkins
 sudo systemctl start jenkins
 ```
 Access: `http://<jenkins-ec2-ip>:8080`
### B. Install Jenkins Plugins- Git Plugin- Pipeline- Docker Pipeline- SonarQube Scanner- Nexus Artifact Uploader- Credentials Binding Plugin--
## 6. Jenkins Configuration
 ### A. SonarQube
 1. Go to **Manage Jenkins > Configure System**
 2. Under **SonarQube Servers**:
   - Name: `SonarQube`
   - Server URL: `http://<sonarqube-ec2-ip>:9000`
   - Token: Add as Jenkins Credential (ID: `sonar-auth`)
 ### B. Nexus Docker Registry
 1. Go to **Manage Jenkins > Credentials**
 2. Add new **Username/Password**:
   - ID: `docker-creds`
   - Username: Nexus login
   - Password: Nexus password
 3. Docker push URL: `http://<nexus-ec2-ip>:8082`--
## 7. Jenkinsfile Pipeline Stages
 Include these stages:- Continues Download- Continues Integration- - SonarQube Scan- Docker Build- Docker Push to Nexus
 Use environment variables to link to SonarQube and Nexus hosts.--
Now your Jenkins server on EC2 is fully integrated with SonarQube and Nexus, all running on separate AWS EC2 instances.