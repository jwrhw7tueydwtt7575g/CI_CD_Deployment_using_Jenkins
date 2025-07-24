# Jenkins with Docker and AWS Deployment Overview

This README provides a comprehensive guide to setting up a Jenkins instance with Docker-in-Docker (DinD), installing necessary tools, building and deploying Docker images, and integrating AWS CLI for deployment.

---

## 📦 Dockerfile

Use the Jenkins image as the base image:

```Dockerfile
FROM jenkins/jenkins:lts

# Switch to root user to install dependencies
USER root

# Install prerequisites and Docker
RUN apt-get update -y && \
    apt-get install -y apt-transport-https ca-certificates curl gnupg software-properties-common && \
    curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add - && \
    echo "deb [arch=amd64] https://download.docker.com/linux/debian bullseye stable" > /etc/apt/sources.list.d/docker.list && \
    apt-get update -y && \
    apt-get install -y docker-ce docker-ce-cli containerd.io && \
    apt-get clean

# Add Jenkins user to the Docker group (create if it doesn't exist)
RUN groupadd -f docker && \
    usermod -aG docker jenkins

# Create Docker directory and volume for DinD
RUN mkdir -p /var/lib/docker
VOLUME /var/lib/docker

# Switch back to Jenkins user
USER jenkins
🔧 Build Docker Image
bash
Copy
Edit
docker build -t jenkins-dind .
🛠️ Kill Process on Port 8080 (Windows)
bash
Copy
Edit
netstat -ano | find ":8080"
taskkill /PID <PID> /F
🚀 Run Jenkins with Docker-in-Docker
bash
Copy
Edit
docker run -d --name jenkins-dind ^
  --privileged ^
  -p 8080:8080 -p 50000:50000 ^
  -v //var/run/docker.sock:/var/run/docker.sock ^
  -v jenkins_home:/var/jenkins_home ^
  jenkins-dind
🐍 Install Python and Tools in the Container
bash
Copy
Edit
docker exec -u root -it jenkins-dind apt update -y
docker exec -u root -it jenkins-dind apt install -y python3
docker exec -u root -it jenkins-dind python3 --version
docker exec -u root -it jenkins-dind ln -s /usr/bin/python3 /usr/bin/python
docker exec -u root -it jenkins-dind python --version
docker exec -u root -it jenkins-dind apt install -y python3-pip
docker exec -u root -it jenkins-dind apt install -y python3-venv
🧪 Jenkinsfile
groovy
Copy
Edit
pipeline {
    agent any

    stages {
        stage('Build Docker image') {
            steps {
                script {
                    echo 'Build Docker image...'
                    docker.build("mlops")
                }
            }
        }
    }
}
🔍 Trivy FileSystem Scan
Install Trivy:

bash
Copy
Edit
docker exec -u root -it jenkins-dind bash
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin v0.57.1
👥 Add Jenkins User to Docker Group
bash
Copy
Edit
docker exec -u root -it jenkins-dind bash
groupadd docker
usermod -aG docker jenkins
usermod -aG root jenkins
🐍 Sample Dockerfile for Python Application
Dockerfile
Copy
Edit
FROM python:slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

RUN apt-get update && \
    apt-get install -y --no-install-recommends libgomp1 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

COPY . .

RUN pip install --no-cache-dir -e .

RUN python main.py

EXPOSE 5000

CMD ["python", "application.py"]
☁️ AWS Deployment
Configure AWS CLI
bash
Copy
Edit
aws configure
Install AWS CLI in the Container
bash
Copy
Edit
docker exec -u root -it jenkins-dind bash
aws --version
apt update && apt upgrade -y
apt-get install curl unzip sudo -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
exit
docker restart jenkins-dind
🔄 Update ECS Service
bash
Copy
Edit
sh "aws ecs update-service --cluster <cluster-name> --service <service-name> --force-new-deployment"
✅ Summary
This setup provides:

A Jenkins environment integrated with Docker.

Trivy for filesystem scanning.

AWS CLI for deployment.
