# 🚀 DevOps Project 003 
# Advanced CI/CD Pipeline

<div align="center">

![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Maven](https://img.shields.io/badge/Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)
![SonarCloud](https://img.shields.io/badge/SonarCloud-F3702A?style=for-the-badge&logo=sonarcloud&logoColor=white)
![Nexus](https://img.shields.io/badge/Nexus-1B1C30?style=for-the-badge&logo=sonatype&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)

**Advanced CI/CD Pipeline with Terraform, Ansible, Jenkins Master-Agent, SonarCloud & Nexus**

</div>

---

## 📋 Architecture Overview

```
Developer → GitHub Push
                ↓
        Jenkins Master (EC2)
                ↓ (SSH)
        Jenkins Agent (EC2)
                ↓
        Maven Build
                ↓
        SonarCloud Analysis
                ↓
        Nexus Repository Deploy
```

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| **Terraform** | Infrastructure provisioning (VPC, EC2, Security Groups) |
| **Ansible** | Configuration management (Jenkins setup) |
| **Jenkins Master** | CI/CD orchestration |
| **Jenkins Agent** | Build execution (Maven) |
| **Maven** | Java application build tool |
| **SonarCloud** | Code quality & security analysis |
| **Nexus Repository** | Artifact storage (Maven releases) |
| **AWS EC2** | Cloud compute instances |
| **AWS VPC** | Network isolation |

---

## 🏗️ Infrastructure Setup

### EC2 Instances Created via Terraform

| Server | Instance Type | Purpose |
|--------|--------------|---------|
| Terraform Controller | t2.micro | Infrastructure management |
| Ansible Controller | t2.micro | Configuration management |
| Jenkins Master | t2.medium | CI/CD orchestration |
| Jenkins Agent | t2.medium | Build execution |

---

## 📁 Project Structure

```
DevOps-Project-003/
├── terraform/
│   ├── main.tf          ← VPC, Subnet, EC2, Security Groups
│   ├── variables.tf     ← Input variables
│   └── outputs.tf       ← Output values (IPs)
├── ansible/
│   ├── inventory.ini    ← Jenkins Master & Agent hosts
│   └── jenkins-setup.yml ← Ansible Playbook
├── src/
│   └── main/java/com/devops/
│       └── App.java     ← Java application
├── pom.xml              ← Maven build config + Nexus deploy
├── Jenkinsfile          ← CI/CD Pipeline definition
└── README.md
```

---

## 🚀 Implementation Steps

### STEP 1 — Terraform Controller Setup

```bash
# Install tools
apt update -y
apt install -y gnupg software-properties-common curl unzip

# Install Terraform
curl -fsSL https://apt.releases.hashicorp.com/gpg | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list
apt update && apt install terraform -y

# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip && ./aws/install

# Install Ansible
apt install ansible -y

# Verify
terraform --version && aws --version && ansible --version
```

### STEP 2 — AWS Configure

```bash
aws configure
# AWS Access Key ID: YOUR_KEY
# AWS Secret Access Key: YOUR_SECRET
# Default region: us-east-1
# Output format: json

# Verify
aws sts get-caller-identity
```

### STEP 3 — Terraform Files

**variables.tf**
```hcl
variable "region" { default = "us-east-1" }
variable "vpc_cidr" { default = "10.0.0.0/16" }
variable "subnet_cidr" { default = "10.0.1.0/24" }
variable "ami_id" { default = "ami-0c7217cdde317cfec" }
variable "key_name" { default = "YOUR-KEY-PAIR-NAME" }
```

**main.tf** — Creates VPC, Subnet, IGW, Route Table, Security Group, and 3 EC2 instances (Ansible Controller, Jenkins Master, Jenkins Agent)

**outputs.tf** — Outputs public IPs of all instances

### STEP 4 — Terraform Init & Apply

```bash
cd /home/ubuntu/DevOps-Project-006/terraform

terraform init
terraform plan
terraform apply -auto-approve
```

**Output:**
```
ansible_controller_ip = "x.x.x.x"
jenkins_master_ip     = "x.x.x.x"
jenkins_agent_ip      = "x.x.x.x"
vpc_id                = "vpc-xxxxxxxxx"
```

### STEP 5 — SSH Key Setup (Ansible Controller)

```bash
# Copy .pem file to Ansible Controller
scp -i C:\Users\USERNAME\Downloads\KEY.pem C:\Users\USERNAME\Downloads\KEY.pem ubuntu@ANSIBLE_IP:/home/ubuntu/

# SSH into Ansible Controller
ssh -i KEY.pem ubuntu@ANSIBLE_IP

# Set permissions
chmod 400 /home/ubuntu/KEY.pem

# Test connections
ssh -i /home/ubuntu/KEY.pem ubuntu@JENKINS_MASTER_IP "echo Connected!"
ssh -i /home/ubuntu/KEY.pem ubuntu@JENKINS_AGENT_IP "echo Connected!"
```

### STEP 6 — Ansible Inventory & Playbook

**inventory.ini**
```ini
[jenkins_master]
JENKINS_MASTER_IP ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/KEY.pem

[jenkins_agent]
JENKINS_AGENT_IP ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/KEY.pem

[all:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

**jenkins-setup.yml** — Installs Java 21, Jenkins WAR, systemd service on Master; Java 21, Maven, Docker, Git on Agent

```bash
# Test connectivity
ansible all -i inventory.ini -m ping

# Run playbook
ansible-playbook -i inventory.ini jenkins-setup.yml
```

> ⚠️ **Important Fix:** Ansible playbook installs Java 17 by default but Jenkins 2.555+ requires Java 21. Always use `openjdk-21-jre` in playbook.

### STEP 7 — Jenkins Master Access

```bash
# Get initial password
cat /var/jenkins_home/secrets/initialAdminPassword
```

Open: `http://JENKINS_MASTER_IP:8080`

**Plugins to install:**
- SSH Agent
- SonarQube Scanner

### STEP 8 — Jenkins Agent SSH Connection

```bash
# On Ansible Controller — generate PEM format RSA key
ssh-keygen -t rsa -b 4096 -f ~/.ssh/jenkins_agent -N "" -m PEM

# Add public key to Jenkins Agent
PUB_KEY=$(cat ~/.ssh/jenkins_agent.pub)
ssh -i /home/ubuntu/KEY.pem ubuntu@JENKINS_AGENT_IP "echo '$PUB_KEY' >> ~/.ssh/authorized_keys"
```

**Jenkins UI:** Manage Jenkins → Credentials → Add SSH Username with private key

**Node Configuration:**
- Launch method: `Launch agents via SSH`
- Host: `JENKINS_AGENT_IP`
- Credentials: `jenkins-agent-ssh`
- Host Key Verification: `Non verifying`

> ⚠️ **Important Fix:** Use RSA key with `-m PEM` flag. Ed25519 keys cause "PEM problem: unknown type" error with Jenkins SSH plugin.

### STEP 9 — SonarCloud Setup

1. Login at sonarcloud.io with GitHub
2. Create organization: `YOUR_GITHUB_USERNAME`
3. Import repository: `DevOps-Project-003`
4. Generate token: My Account → Security → Generate Token

**Jenkins:** Manage Jenkins → System → SonarQube servers:
- Name: `SonarCloud`
- URL: `https://sonarcloud.io`
- Token: `sonar-token`

### STEP 10 — Nexus Setup (Jenkins Agent)

```bash
docker run -d \
  --name nexus \
  -p 8081:8081 \
  sonatype/nexus3

# Get admin password
docker exec nexus cat /nexus-data/admin.password
```

Open: `http://JENKINS_AGENT_IP:8081`

**Maven settings.xml** (on Jenkins Agent):
```xml
<settings>
  <servers>
    <server>
      <id>nexus-releases</id>
      <username>admin</username>
      <password>YOUR_NEXUS_PASSWORD</password>
    </server>
  </servers>
</settings>
```

> ⚠️ **Important Fix:** Create `/home/ubuntu/.m2/repository` with `chmod 777` before running pipeline.

### STEP 11 — Java Maven App

**pom.xml** — Includes Maven compiler (Java 11) and Nexus distribution management

**App.java:**
```java
package com.devops;

public class App {
    public static void main(String[] args) {
        System.out.println("Hello from DevOps Project 06!");
    }
}
```

### STEP 12 — Jenkinsfile

```groovy
pipeline {
    agent { label 'jenkins-agent' }
    
    environment {
        SONAR_PROJECT_KEY = 'YOUR_ORG_DevOps-Project-003'
        SONAR_ORG = 'YOUR_ORG'
        NEXUS_URL = 'http://JENKINS_AGENT_IP:8081'
    }
    
    stages {
        stage('Clone Repository') {
            steps { checkout scm }
        }
        stage('Build with Maven') {
            steps { sh 'mvn clean package -DskipTests' }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.organization=${SONAR_ORG}'
                }
            }
        }
        stage('Deploy to Nexus') {
            steps { sh 'mvn deploy -DskipTests' }
        }
    }
}
```

### STEP 13 — Pipeline Job

Jenkins → New Item → Pipeline:
- Definition: `Pipeline script from SCM`
- SCM: `Git`
- Repository URL: `https://github.com/hmurafique/DevOps-Project-003.git`
- Branch: `*/main`
- Script Path: `Jenkinsfile`

---

## ✅ Pipeline Results

```
✅ Clone Repository    — SUCCESS
✅ Build with Maven    — SUCCESS  
✅ SonarQube Analysis  — SUCCESS
✅ Deploy to Nexus     — SUCCESS
```

**Nexus Artifact:**
```
com → devops → devops-java-app → 1.0.0 ✅
```

---

## 🐛 Common Issues & Fixes

| Issue | Fix |
|-------|-----|
| Jenkins fails with Java 17 | Install `openjdk-21-jre` on Jenkins Master |
| SSH Agent key rejected (PEM unknown type) | Use `ssh-keygen -t rsa -m PEM` instead of ed25519 |
| Maven: Cannot create local repository | `chmod 777 /home/ubuntu/.m2` |
| Ansible ping fails | Install ansible: `apt install ansible -y` |
| Jenkins Agent offline | Check SSH key in authorized_keys on Agent |

---

## 🧹 Cleanup

```bash
# Delete ECS services, Nexus container
docker stop nexus && docker rm nexus

# Terraform destroy
cd /home/ubuntu/DevOps-Project-003/terraform
terraform destroy -auto-approve
```

---

## 📚 What I Learned

- **Terraform** — Infrastructure as Code for AWS VPC + EC2 provisioning
- **Ansible** — Automated server configuration with playbooks
- **Jenkins Master-Agent** — Distributed build architecture
- **SonarCloud** — Code quality gates in CI/CD pipeline
- **Nexus** — Artifact repository management
- **Maven** — Java build lifecycle and dependency management

---

## 👤 Author

**Hafiz Muhammad Umar Rafique**
- GitHub: [@hmurafique](https://github.com/hmurafique)
- Project: DevOps Projects Portfolio

---

<div align="center">
⭐ Star this repo if you found it helpful!
</div>

