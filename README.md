# **Devops-projects-Two-Tier-Flask-App**

> Production-like 2-tier web application (Flask + MySQL) deployed using Docker & Docker Compose on an AWS EC2 instance, fully automated with a declarative Jenkins CI/CD pipeline (master + dev agent setup).

---

## **📌 Table of Contents**

- [Project Overview](#project-overview)  
- [Architecture](#architecture)  
- [Repository Structure](#repository-structure)  
- [Prerequisites](#prerequisites)  
- [Quick Start — Local Run](#quick-start--local-run)  
- [Deploy to AWS EC2](#deploy-to-aws-ec2)  
- [CI/CD — Jenkins Pipeline](#cicd--jenkins-pipeline)  
- [Environment & Secrets](#environment--secrets)  
- [Rollback & Troubleshooting](#rollback--troubleshooting)  
- [Contributing](#contributing) 

---

## **Project Overview**

This project demonstrates a **complete DevOps workflow** for deploying a 2-tier web application:

- **Flask Web App** (`app.py` + templates)
- **MySQL Database** with seed data (`message.sql`)
- **Docker & Docker Compose** for consistent packaging and runtime
- **AWS EC2** as deployment target  
- **Full CI/CD pipeline using Jenkins Declarative syntax**
  - Jenkins **master** for orchestration  
  - Jenkins **dev agent** for deployments  
  - Automatic build + deploy on GitHub commits

This showcases real-world DevOps skills suitable for **production workflows, internships, or portfolio demonstration**.

---

## **Architecture**

**GitHub Repo → Webhook → Jenkins Master
→ Build & Test
→ Deploy via Dev Agent
→ AWS EC2
→ Docker Compose (Flask + MySQL)**


### Key Design Principles

- Clear separation: CI **(master)** and CD **(dev agent)**
- Infrastructure as code: **Docker** + **Jenkinsfile**
- Automated deployments with **zero** manual intervention
- **Environment-driven** config for production

---

## **Repository Structure**


├── **app.py** # Flask application                       
├── **Dockerfile** # Flask Docker image        
├── **docker-compose.yml** # App + MySQL containers  
├── **Jenkinsfile** # Declarative CI/CD pipeline  
├── **requirement.txt** # Python dependencies  
├── **message.sql** # MySQL seed data
├── **templates/** # HTML templates for Flask    
└── **README.md** # Documentation


---

## **Prerequisites**

To **run** or **deploy** this project, ensure you have:

### Local / Development
- **Docker Engine** installed  
- **Docker Compose** installed  
- **Python 3.8+** (optional for manual testing)

### CI/CD / AWS

- **Jenkins** server accessible from **GitHub**
- Jenkins **dev agent** with Docker installed
- **AWS EC2** instance (Ubuntu recommended)
- **SSH** access to **EC2** (key pair)
- Jenkins credentials for **SSH** / registry (if used)

---

## **Quick Start — Local Run**

### 1. Clone repository
```bash
git clone https://github.com/anshdeploys/Devops-projects-Two-Tier-Flask-App.git
cd Devops-projects-Two-Tier-Flask-App
```

### 2. Run with Docker Compose
```bash
docker compose up --build -d
```

### 3. Access the app
```bash
http://localhost:5000
```

### 4. Stop the app
```bash
docker compose down
```

## **Deploy to AWS EC2**

### 1. Install Docker & Compose on EC2
```bash
sudo apt update
sudo apt-get install docker.io
sudo usermod -aG docker ubuntu
```

#### Install Compose:

#### (Option 1) — Install the Official Binary (Ensures Latest Version)

```bash
sudo curl -L https://github.com/docker/compose/releases/download/v2.6.0/docker-compose-linux-x86_64 \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

#### (Option 2) — Install via APT (Simplest for Ubuntu EC2)
```bash
sudo apt-get update
sudo apt-get install docker-compose-v2
```
### 2. Pull code to EC2
```bash
git clone https://github.com/anshdeploys/Devops-projects-Two-Tier-Flask-App.git
cd Devops-projects-Two-Tier-Flask-App
```
### 3. Deploy
```bash
docker compose up --build -d flask-app
```
## **CI/CD — Jenkins Pipeline**

The Jenkinsfile implements a full **declarative pipeline** with the following stages:

**✔️ Checkout**

Uses checkout scm to pull the code automatically.

**✔️ Build & Test**

Builds Docker images and runs optional tests.

**✔️ Deployment** (using dev agent)

Jenkins dev agent SSHs into EC2 and runs:
```bash
git pull && docker compose up -d --build
```

**✔️ Post Actions**

Success/failure notifications.

## Example Pipeline Snippet

pipeline {
  agent none
  
  stages {
    stage('Checkout') {
      agent { label 'master' }
      steps {
        checkout scm
      }
    }

    stage('Build & Test') {
      agent { label 'build' }
      steps {
        sh 'docker compose build --pull'
        sh 'pytest -q || true'
      }
    }

    stage('Deploy to EC2') {
      agent { label 'dev' }
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh',
                                           keyFileVariable: 'KEY',
                                           usernameVariable: 'EC2_USER')]) {
          sh '''
            ssh -i $KEY $EC2_USER@EC2_HOST '
              cd /path/to/repo &&
              git pull origin main &&
              docker compose up -d --build
            '
          '''
        }
      }
    }
  }

  post {
    success { echo '**Deployment Successful!**' }
    failure { echo '**Deployment Failed!**' }
  }
}

## **Environment & Secrets**

Your application requires sensitive values like database passwords, usernames, and API keys.  
These should **never be committed to GitHub**. Use one of the following methods to store secrets securely:

### 1. Jenkins Credentials Store
- Store secrets like **SSH keys, passwords, or tokens** securely inside Jenkins.  
- Jenkins encrypts them and they can be accessed in the pipeline using `withCredentials`.

### 2. AWS Systems Manager Parameter Store
- Store secrets on **AWS for EC2 or production environments.**  
- Supports **encryption** and secure retrieval.  
- Example command to **retrieve** a parameter:  
```bash
aws ssm get-parameter --name "/app/prod/db_password" --with-decryption
```

### 3. Local .env File on EC2 (NOT committed)

- Create a **.env** file on your EC2 instance for Docker Compose:
```bash
.env
  MYSQL_ROOT_PASSWORD=secure-pass
  MYSQL_DATABASE=appdb
  MYSQL_USER=appuser
  MYSQL_PASSWORD=strongpass
```
- Docker Compose **automatically** loads these environment variables.

- **Important:** Do not push .env to GitHub.


## **Rollback & Troubleshooting**
### Rollback deployment
```bash
docker compose up -d --no-build
```

## Check logs
```bash
docker compose logs -f
```
| Error                     | Fix                                         |
| ------------------------- | ------------------------------------------- |
| `no such service`         | Service name mismatch in docker-compose.yml |
| Docker buildx errors      | Remove buildx plugin or reinstall docker    |
| Jenkins cannot run Docker | Add Jenkins user to docker group            |

## **Contributing**

#### 1. Fork the repo

#### 2. Create a branch

#### 3. Commit and push

#### 4. Open a Pull Request

All contributions are welcome!