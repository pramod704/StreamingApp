pipeline {
  agent any
  environment {
    AWS_REGION = 'us-east-1'
    ECR_REGISTRY = '506189153515.dkr.ecr.us-east-1.amazonaws.com'
  }
  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/UnpredictablePrashant/StreamingApp.git']]])
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
        // Use the official AWS CLI Docker image to generate ECR login password
        withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
          sh '''
          docker run --rm -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
            amazon/aws-cli:2.11.4 ecr get-login-password --region $AWS_REGION | \
            docker login --username AWS --password-stdin $ECR_REGISTRY
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
