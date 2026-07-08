pipeline {
  agent any
  environment {
    AWS_REGION = 'eu-north-1'
    ECR_REGISTRY = '506189153515.dkr.ecr.eu-north-1.amazonaws.com'
  }
  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/pramod704/StreamingApp.git']]])
      }
    }

    stage('Build Frontend') {
      steps {
        dir('frontend') {
          sh 'docker build -t frontend:latest .'
        }
      }
    }

    stage('Build Backend Services') {
      steps {
        dir('backend/authService') { sh 'docker build -t backend-auth:latest .' }
        dir('backend/streamingService') { sh 'docker build -t backend-streaming:latest .' }
        dir('backend/adminService') { sh 'docker build -t backend-admin:latest .' }
        dir('backend/chatService') { sh 'docker build -t backend-chat:latest .' }
      }
    }

    stage('Login to ECR') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
          sh '''
          aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
          aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
          aws configure set region $AWS_REGION
          aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY
          '''
        }
      }
    }

    stage('Tag & Push') {
      steps {
        sh "docker tag frontend:latest ${ECR_REGISTRY}/frontend:latest"
        sh "docker push ${ECR_REGISTRY}/frontend:latest"

        sh "docker tag backend-auth:latest ${ECR_REGISTRY}/backend-auth:latest"
        sh "docker push ${ECR_REGISTRY}/backend-auth:latest"

        sh "docker tag backend-streaming:latest ${ECR_REGISTRY}/backend-streaming:latest"
        sh "docker push ${ECR_REGISTRY}/backend-streaming:latest"

        sh "docker tag backend-admin:latest ${ECR_REGISTRY}/backend-admin:latest"
        sh "docker push ${ECR_REGISTRY}/backend-admin:latest"

        sh "docker tag backend-chat:latest ${ECR_REGISTRY}/backend-chat:latest"
        sh "docker push ${ECR_REGISTRY}/backend-chat:latest"
      }
    }
  }
  post {
    always {
      sh 'docker image prune -f || true'
    }
  }
}
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
