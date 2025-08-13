pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        AWS_ACCOUNT_ID = "992382545251"
        ECR_REPO = "yuvalz-repo"
        IMAGE_TAG = "main-${BUILD_NUMBER}"
        ECR_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
        EC2_HOST = "54.175.24.85"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SixteZuki/CDexercise.git'
            }
        }

        stage('Build Docker image') {
            steps {
                dir('CDexercise') {
                    sh 'docker build -t ${ECR_REPO}:${IMAGE_TAG} -f Dockerfile .'
                }
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Tag & Push to ECR') {
            steps {
                sh '''
                docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URL}:${IMAGE_TAG}
                docker push ${ECR_URL}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ec2-user@${EC2_HOST} << 'EOF'
                docker pull ${ECR_URL}:${IMAGE_TAG}
                docker stop flask-app || true
                docker rm flask-app || true
                docker run -d --name flask-app -p 5000:5000 ${ECR_URL}:${IMAGE_TAG}
                EOF
                '''
            }
        }
    }
}
