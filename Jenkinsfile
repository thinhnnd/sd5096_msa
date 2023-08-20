pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }

    environment {
        AWS_REGION = 'ap-southeast-1'
        BACKEND_REPO = 'practical-devops-erc/backend'
        FRONTEND_REPO = 'practical-devops-erc/frontend'
        ECR_REGISTRY = '309276322609.dkr.ecr.ap-southeast-1.amazonaws.com/practical-devops-erc'
    }

    stages {
         stage('Clone repository') { 
            steps { 
                script{
                checkout scm
                }
            }
        }
        
        stage('Build Backend Image') {
            steps {
                script {
                    // Build backend Docker image
                    def backendImage = docker.build(context: 'src/backend', dockerfile: 'src/backend/Dockerfile', tag: "${BACKEND_REPO}:${env.BUILD_NUMBER}")
                    // Push backend image to ECR
                    withAWS(region: AWS_REGION, credentials: 'aws-credential') {
                        docker.withRegistry(ECR_REGISTRY, 'ecr:ap-southeast-1:my-aws-credentials') {
                            backendImage.push()
                        }
                    }
                }
            }
        }
        
        stage('Build Frontend Image') {
            steps {
                script {
                    // Build frontend Docker image
                    def frontendImage = docker.build(context: 'src/frontend', dockerfile: 'src/frontend/Dockerfile', tag: "${FRONTEND_REPO}:${env.BUILD_NUMBER}")
                    // Push frontend image to ECR
                    withAWS(region: AWS_REGION, credentials: 'aws-credential') {
                        docker.withRegistry(ECR_REGISTRY, 'ap-southeast-1:my-aws-credentials') {
                            frontendImage.push()
                        }
                    }
                }
            }
        }   
    }

    post {
        always {
            cleanWs()
        }
    }
}