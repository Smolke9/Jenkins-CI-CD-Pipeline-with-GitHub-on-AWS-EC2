# Jenkins CI/CD Pipeline with GitHub on AWS EC2

## Overview
Integrating Jenkins with GitHub on an AWS EC2 instance automates the build, test, and deployment pipeline using Docker, Jenkins, and GitHub.

---

## Prerequisites
- AWS account
- GitHub repository (sample project)
- Docker Hub account
- Basic knowledge of EC2, Git, Docker

---

## Step 1: Launch an EC2 Instance
1. Log into AWS Console
2. Navigate to EC2 > Launch Instance
3. Choose Ubuntu 22.04 LTS AMI
4. Select instance type: t2.micro
5. Create key pair (.pem)
6. Configure security group:
   - SSH (22) – My IP
   - HTTP (80) – Anywhere
   - HTTPS (443) – Anywhere
   - Custom TCP (8080) – My IP
7. Launch the instance

---

## Step 2: Set Up Docker and Jenkins

### Connect to EC2
```bash
ssh -i "your-key.pem" ubuntu@<public-ip>
```

### Install Docker
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
```

### Install Java
```bash
sudo apt install openjdk-17-jre -y
```

### Install Jenkins
```bash
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
```

### Start Jenkins
```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

## Step 3: Access Jenkins
1. Visit: http://<public-ip>:8080
2. Get initial admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
3. Install Suggested Plugins
4. Create Admin User

---

## Step 4: Prepare Project

### Clone Sample Project
```bash
git clone https://github.com/Smolke9
cd django-notes-app
```

### Build & Run Docker Container
```bash
docker build -t notes-app .
docker run -d -p 8000:8000 notes-app
```

---

## Step 5: Create Jenkins Pipeline Job
1. Jenkins Dashboard → New Item → Pipeline → OK
2. Under Pipeline Script:

```groovy
pipeline {
    agent any

    stages {
        stage("Clone Code") {
            steps {
                git url: 'https://github.com/', branch: 'main'
            }
        }

        stage("Build Image") {
            steps {
                sh 'docker build -t my-note-app .'
            }
        }

        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerHub', usernameVariable: 'dockerHubUser', passwordVariable: 'dockerHubPass')]) {
                    sh 'docker tag my-note-app ${dockerHubUser}/my-note-app:latest'
                    sh 'docker login -u ${dockerHubUser} -p ${dockerHubPass}'
                    sh 'docker push ${dockerHubUser}/my-note-app:latest'
                }
            }
        }

        stage("Deploy") {
            steps {
                sh 'docker-compose down || true'
                sh 'docker-compose up -d'
            }
        }
    }
}
```

3. Enable GitHub hook trigger for GITScm polling

---

## Step 6: Add Docker Hub Credentials
1. Jenkins Dashboard → Manage Jenkins → Credentials → Global → Add Credentials
2. Select "Username with password"
3. Credential ID: `dockerHub`

---

## Step 7: Configure GitHub Webhook
1. Go to GitHub repo → Settings → Webhooks
2. Add Webhook:
   - URL: http://<EC2-IP>:8080/github-webhook/
   - Content type: application/json
   - Trigger: Just the push event

---

## Step 8: Build & Deploy
1. Trigger build manually: **Build Now**
2. Or push code to GitHub to trigger via webhook
3. Monitor console output
4. App should be accessible at: http://<EC2-IP>:8000

---

## ✅ Result
Fully automated CI/CD from GitHub to Dockerized deployment on EC2 using Jenkins!

