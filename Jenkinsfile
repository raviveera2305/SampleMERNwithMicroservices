pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '526362561261.dkr.ecr.ap-south-1.amazonaws.com'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ECR_REGISTRY
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    docker build \
                        -t $ECR_REGISTRY/hello-service:latest \
                        ./backend/helloService

                    docker build \
                        -t $ECR_REGISTRY/profile-service:latest \
                        ./backend/profileService

                    docker build \
                        -t $ECR_REGISTRY/frontend:latest \
                        ./frontend
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                    docker push $ECR_REGISTRY/hello-service:latest
                    docker push $ECR_REGISTRY/profile-service:latest
                    docker push $ECR_REGISTRY/frontend:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'All Docker images were successfully pushed to Amazon ECR.'
        }

        failure {
            echo 'Jenkins pipeline failed. Check the console output.'
        }
    }
}