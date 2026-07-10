pipeline {
    agent {
        label 'linux'
    }

    environment {
    APP_NAME = "helm"  
    ECR_REGISTRY = " 486517829811.dkr.ecr.ap-south-1.amazonaws.com"
    ECR_REPO = "${ECR_REGISTRY}/${APP_NAME}"
    IMAGE_TAG = "${BUILD_NUMBER}"
    KUBE_NAMESPACE = "helm-deployment"
    }

    stages {
        stage('Clone Git Repo') {
            steps {
                git 'https://github.com/Ravichandu-git/docker-spring-boot.git'
            }
        }

        stage('Build Java App') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
                sh "docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_REPO}:latest"
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    // Ensure the ECR repository exists
                    sh """
                    aws ecr describe-repositories --repository-names ${APP_NAME} --region ap-south-1 || \
                    aws ecr create-repository --repository-name ${APP_NAME} --region ap-south-1
                    """

                    // Login to ECR
                    sh "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${ECR_REGISTRY}"

                    // Push Docker images
                    sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
                    sh "docker push ${ECR_REPO}:latest"
                }
            }
        }
        
        stage('Deploy to EKS using Helm') {
            steps {
                script {
                    sh """
            aws eks --region ap-south-1 update-kubeconfig --name demo-cluster
            helm upgrade --install first ./mychart \
                --namespace ${KUBE_NAMESPACE} \
                --create-namespace \
                --set image.repository=${ECR_REPO} \
                --set image.tag=${IMAGE_TAG}
            """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful: ${APP_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "Deployment failed"
        }
    }
}
