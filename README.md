# End-to-End CI/CD (Java & Python) --- Jenkins, Docker, Kubernetes, EKS

## Overview

This project demonstrates a complete CI/CD pipeline: GitHub → Jenkins →
Build → Docker → Kubernetes (EKS)

## Prerequisites

-   AWS EC2 (t2.medium)
-   Java 17
-   Jenkins
-   Docker
-   kubectl
-   Helm

## Jenkins Setup (Amazon Linux)

``` bash
sudo dnf install java-17-amazon-corretto -y
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo dnf install jenkins -y
sudo systemctl enable --now jenkins
sudo dnf install docker -y
sudo usermod -aG docker jenkins
sudo systemctl enable --now docker
```

## Java Jenkins Pipeline

``` groovy
pipeline {
    agent any
    tools {
        maven 'Maven3'
        jdk 'JDK17'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/project/repo.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                pkill java || true
                nohup java -jar target/*.jar > app.log 2>&1 &
                '''
            }
        }
    }
}
```

## Python Jenkins Pipeline

``` groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/project/python-app.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh '''
                python3 -m venv venv
                source venv/bin/activate
                pip install -r requirements.txt
                '''
            }
        }
        stage('Run App') {
            steps {
                sh '''
                source venv/bin/activate
                python app.py &
                '''
            }
        }
    }
}
```

## Dockerfiles

Java:

``` dockerfile
FROM eclipse-temurin:17-jdk-alpine
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```

Python:

``` dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
EXPOSE 5000
CMD ["python","app.py"]
```

## Kubernetes Deployment

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: repo/app:v1
        ports:
        - containerPort: 8080
```

Apply:

``` bash
kubectl apply -f deployment.yaml
```

## Helm Upgrade

``` bash
helm upgrade --install app chart/ --set image.tag=v2
```

## EKS Deployment

``` bash
aws eks update-kubeconfig --region ap-south-1 --name mycluster
docker push <ECR-URL>
helm upgrade --install app chart/
```

## Interview Explanation

CI/CD automatically builds, packages and deploys applications. Pipeline
stages remain same for Java and Python --- only build commands change.
