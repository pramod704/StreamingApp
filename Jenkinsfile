pipeline {
  agent any

  environment {
    AWS_REGION = 'us-east-1'
    ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Images') {
      steps {
        sh '''
          docker build -t streaming-auth:latest ./backend/authService
          docker build -t streaming-streaming:latest ./backend/streamingService
          docker build -t streaming-admin:latest ./backend/adminService
          docker build -t streaming-chat:latest ./backend/chatService
          docker build -t streaming-frontend:latest ./frontend
        '''
      }
    }

    stage('Login to ECR') {
      steps {
        sh '''
          aws ecr get-login-password --region "$AWS_REGION" \
            | docker login --username AWS --password-stdin "$ECR_REGISTRY"
        '''
      }
    }

    stage('Push Images to ECR') {
      steps {
        sh '''
          docker tag streaming-auth:latest "$ECR_REGISTRY/streaming-auth:latest"
          docker tag streaming-streaming:latest "$ECR_REGISTRY/streaming-streaming:latest"
          docker tag streaming-admin:latest "$ECR_REGISTRY/streaming-admin:latest"
          docker tag streaming-chat:latest "$ECR_REGISTRY/streaming-chat:latest"
          docker tag streaming-frontend:latest "$ECR_REGISTRY/streaming-frontend:latest"

          docker push "$ECR_REGISTRY/streaming-auth:latest"
          docker push "$ECR_REGISTRY/streaming-streaming:latest"
          docker push "$ECR_REGISTRY/streaming-admin:latest"
          docker push "$ECR_REGISTRY/streaming-chat:latest"
          docker push "$ECR_REGISTRY/streaming-frontend:latest"
        '''
      }
    }
  }
}
