pipeline {
    agent any
    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "992382545251.dkr.ecr.us-east-1.amazonaws.com/yuvalz-repo"
        EC2_HOST = "ec2-user@54.175.24.85"
        DOCKER_IMAGE = "${ECR_REPO}:${BRANCH_NAME}-${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SixteZuki/CDexercise.git'
            }
        }
        stage('Build Docker image') {
            steps {
                sh "docker build -t ${ECR_REPO}:${BRANCH_NAME}-${BUILD_NUMBER} -f Dockerfile ."
            }
        }
        stage('Login to ECR') {
            steps {
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"
            }
        }
        stage('Tag & Push to ECR') {
            steps {
                sh "docker push ${ECR_REPO}:${BRANCH_NAME}-${BUILD_NUMBER}"
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_HOST} '
                        docker pull ${ECR_REPO}:${BRANCH_NAME}-${BUILD_NUMBER} &&
                        docker stop flask-app || true &&
                        docker rm flask-app || true &&
                        docker run -d --name flask-app -p 5000:5000 ${ECR_REPO}:${BRANCH_NAME}-${BUILD_NUMBER}
                    '
                    """
                }
            }
        }
    }
}
