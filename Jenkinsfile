pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '992382545251'
        AWS_REGION = 'us-east-1'
        ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/yuvalz-repo"
        DOCKER_IMAGE = "${ECR_REPO}:${BRANCH_NAME}-${BUILD_NUMBER}"
        EC2_HOST = 'ec2-user@54.175.24.85'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/SixteZuki/CDexercise.git'
            }
        }

        stage('Build Docker image') {
            steps {
                sh """
                    docker build -t ${DOCKER_IMAGE} \
                    -f CDexercise/Dockerfile CDexercise
                """
            }
        }

        stage('Login to ECR') {
            steps {
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"
            }
        }

        stage('Tag & Push to ECR') {
            steps {
                sh """
                    docker push ${DOCKER_IMAGE}
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_HOST} '
                        docker pull ${DOCKER_IMAGE} &&
                        docker stop flask-app || true &&
                        docker rm flask-app || true &&
                        docker run -d --name flask-app -p 5000:5000 ${DOCKER_IMAGE}
                    '
                """
            }
        }
    }
}
