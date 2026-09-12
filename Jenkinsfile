pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '526362561261.dkr.ecr.ap-south-1.amazonaws.com'
        EKS_CLUSTER = 'mern-eks-cluster'
        K8S_NAMESPACE = 'mern-app'
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

        stage('Deploy to EKS with Helm') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                        --name $EKS_CLUSTER \
                        --region $AWS_REGION

                    helm upgrade --install mern-app ./helm/mern-app \
                        --namespace $K8S_NAMESPACE \
                        --create-namespace \
                        --wait \
                        --timeout 10m

                    kubectl rollout status deployment/frontend \
                        --namespace $K8S_NAMESPACE \
                        --timeout=5m

                    kubectl rollout status deployment/hello-service \
                        --namespace $K8S_NAMESPACE \
                        --timeout=5m

                    kubectl rollout status deployment/profile-service \
                        --namespace $K8S_NAMESPACE \
                        --timeout=5m

                    kubectl rollout status deployment/mongodb \
                        --namespace $K8S_NAMESPACE \
                        --timeout=5m
                '''
            }
        }

        stage('Validate EKS Deployment') {
            steps {
                sh '''
                    kubectl get deployments -n $K8S_NAMESPACE
                    kubectl get pods -n $K8S_NAMESPACE
                    kubectl get svc -n $K8S_NAMESPACE
                    kubectl get hpa -n $K8S_NAMESPACE
                '''
            }
        }
    }

    post {
        success {
            echo 'Images were pushed to ECR and the application was successfully deployed to EKS with Helm.'
        }

        failure {
            echo 'Jenkins pipeline failed. Check the console output.'
        }
    }
}
