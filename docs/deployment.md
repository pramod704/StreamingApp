# Deployment Guide

## Overview
This project can be deployed with Docker, Amazon ECR, Jenkins, and Kubernetes (EKS).

## 1. Build Docker Images
```bash
docker build -t streaming-auth:latest ./backend/authService
docker build -t streaming-streaming:latest ./backend/streamingService
docker build -t streaming-admin:latest ./backend/adminService
docker build -t streaming-chat:latest ./backend/chatService
docker build -t streaming-frontend:latest ./frontend
```

## 2. Push Images to Amazon ECR
```bash
aws configure
aws ecr create-repository --repository-name streaming-auth
aws ecr create-repository --repository-name streaming-streaming
aws ecr create-repository --repository-name streaming-admin
aws ecr create-repository --repository-name streaming-chat
aws ecr create-repository --repository-name streaming-frontend

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```

## 3. Jenkins Pipeline
The repository includes a Jenkinsfile that:
- checks out the source code,
- builds the Docker images,
- logs into ECR,
- pushes the images to ECR.

## 4. Kubernetes / EKS Deployment
```bash
eksctl create cluster --name streaming-cluster --region us-east-1 --nodes 2
helm create streamingapp
helm install streamingapp ./streamingapp
```

## 5. Monitoring and Logging
- Use CloudWatch for metrics and alarms.
- Use `kubectl logs <pod-name>` for pod logs.

## 6. Suggested Architecture
- Frontend: React app
- Backend services: auth, streaming, admin, chat
- Database: MongoDB
- Container registry: Amazon ECR
- CI/CD: Jenkins
- Deployment: EKS + Helm
