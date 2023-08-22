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
        AWS_CREDENTIAL_ID = 'thinhnnd_aws_credential'
        MONGODB_CONNECTION_STRING = credentials('mongodb_connection_string')
        AWS_CREDENTIALS = credentials('thinhnnd_aws_credential')
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

                        def filePath = "./config/config.json"
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
                script {

                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-key', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        sh 'aws eks update-kubeconfig --name eks-cluster'
                        sh 'kubectl rollout restart deployment backend -n eks-ns'
                        sh 'kubectl rollout restart deployment frontend -n eks-ns'                    
                    }

                    echo 'deployed to EKS'
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