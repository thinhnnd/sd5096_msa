pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }

    environment {
        AWS_REGION = 'ap-southeast-1'
        BACKEND_REPO = 'backend'
        FRONTEND_REPO = 'frontend'
        AWS_ACCOUNT_ID = '309276322609'
        AWS_CREDENTIAL_ID = 'thinhnnd_aws_credential',
        MONGODB_CONNECTION_STRING = credentials('mongodb_connection_string')
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

                // cleanup current user docker credentials
                // sh 'rm -f ~/.dockercfg ~/.docker/config.json || true'

                dir('src/backend') {
                    script {
                        echo 'Setup env variable.'

                        def jsonData = [
                            development: [
                                PORT: 5000,
                                MONGODB_URI: "${env.MONGODB_CONNECTION_STRING}"
                            ],
                        ]

                        def filePath = "config/config.json"
                        writeFile file: filePath, text: groovy.json.JsonOutput.toJson(jsonData), overwrite: true

                        echo 'Env variable already setup.'

                        def backendImage = docker.build("${env.BACKEND_REPO}:${env.BUILD_NUMBER}")
                        docker.withRegistry("https://${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.BACKEND_REPO}", "ecr:ap-southeast-1:${env.AWS_CREDENTIAL_ID}") {
                            backendImage.push("${env.BUILD_NUMBER}")
                            backendImage.push("latest")
                        }
                    }
                }
                echo 'done buiding backend image.'
            }
        }
        
        stage('Build Frontend Image') {
            steps {
                echo 'Built front end image'

                dir('src/frontend') {
                    script {
                        def frontendImage = docker.build("${env.FRONTEND_REPO}:${env.BUILD_NUMBER}")
                        docker.withRegistry("https://${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.FRONTEND_REPO}", "ecr:ap-southeast-1:${env.AWS_CREDENTIAL_ID}") {
                            frontendImage.push("${env.BUILD_NUMBER}")
                            frontendImage.push("latest")
                        }
                    }
                }
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