pipeline {
  agent any
  options { timestamps() }
  environment {
    AWS_REGION = 'us-east-1'
    ACCOUNT_ID = '992382545251'
    REPO_NAME  = 'yuvalz-repo'
    ECR_URL    = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
    IMAGE_TAG  = "${BRANCH_NAME}-${BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/SixteZuki/CDexercise.git', branch: 'main', credentialsId: 'github-pat'
      }
    }
    stage('Build Docker image') {
      steps {
        sh """
        docker build -t ${REPO_NAME}:${IMAGE_TAG} .
        """
      }
    }
    stage('Login to ECR') {
      steps {
        sh """
        aws ecr get-login-password --region ${AWS_REGION} \
        | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
        """
      }
    }
    stage('Push to ECR') {
      steps {
        sh """
        docker tag ${REPO_NAME}:${IMAGE_TAG} ${ECR_URL}:${IMAGE_TAG}
        docker push ${ECR_URL}:${IMAGE_TAG}
        docker tag ${REPO_NAME}:${IMAGE_TAG} ${ECR_URL}:latest
        docker push ${ECR_URL}:latest
        """
      }
    }
    stage('Deploy to EC2') {
      steps {
        sh """
        docker pull ${ECR_URL}:latest
        docker rm -f flask || true
        docker run -d --name flask --restart unless-stopped -p 80:5000 ${ECR_URL}:latest
        """
      }
    }
  }
}
