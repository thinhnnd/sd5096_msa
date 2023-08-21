pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }

    environment {
        AWS_REGION = 'ap-southeast-1'
        BACKEND_REPO = 'practical-devops-erc/backend'
        FRONTEND_REPO = 'practical-devops-erc/frontend'
        AWS_ACCOUNT_ID = '309276322609'
        AWS_CREDENTIAL_ID = 'thinhnnd_aws_credential'
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
                echo 'start buiding backend image.'
                dir('src/backend') {
                    script {
                        def backendImage = docker.build("${env.BACKEND_REPO}:${env.BUILD_NUMBER}")
                        docker.withRegistry("https://${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/practical-devops-erc", "${ENV.AWS_CREDENTIAL_ID}") {
                            backendImage.push("${BACKEND_REPO}:${env.BUILD_NUMBER}")
                            backendImage.push("latest")
                        }
                    }
                }
                echo 'done buiding backend image.'

                /* script {
                    // Build backend Docker image
                    def backendImage = docker.build(context: 'src/backend', dockerfile: 'src/backend/Dockerfile', tag: "${BACKEND_REPO}:${env.BUILD_NUMBER}")
                    // Push backend image to ECR
                    withAWS(region: AWS_REGION, credentials: 'aws-credential') {
                        docker.withRegistry(ECR_REGISTRY, 'ecr:ap-southeast-1:my-aws-credentials') {
                            backendImage.push()
                        }
                    }
                } */
            }
        }
        
        stage('Build Frontend Image') {
            steps {
                echo 'Built front end image'

                /* script {
                    // Build frontend Docker image
                    def frontendImage = docker.build(context: 'src/frontend', dockerfile: 'src/frontend/Dockerfile', tag: "${FRONTEND_REPO}:${env.BUILD_NUMBER}")
                    // Push frontend image to ECR
                    withAWS(region: AWS_REGION, credentials: 'aws-credential') {
                        docker.withRegistry(ECR_REGISTRY, 'ap-southeast-1:my-aws-credentials') {
                            frontendImage.push()
                        }
                    }
                }*/
            }
        }   

        stage('Deploy to EKS') {
            steps {
                echo 'deployed to EKS'

                /* script {
                    // Build frontend Docker image
                    def frontendImage = docker.build(context: 'src/frontend', dockerfile: 'src/frontend/Dockerfile', tag: "${FRONTEND_REPO}:${env.BUILD_NUMBER}")
                    // Push frontend image to ECR
                    withAWS(region: AWS_REGION, credentials: 'aws-credential') {
                        docker.withRegistry(ECR_REGISTRY, 'ap-southeast-1:my-aws-credentials') {
                            frontendImage.push()
                        }
                    }
                }*/
            }
        }   
    }

    post {
        always {
            cleanWs()
        }
    }
}