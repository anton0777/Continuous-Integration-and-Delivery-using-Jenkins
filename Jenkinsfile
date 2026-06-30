pipeline {
    agent any
    
    environment {
        APP_PORT = "${BRANCH_NAME == 'main' ? '3000' : '3001'}"
        IMAGE_NAME = "${BRANCH_NAME == 'main' ? 'anton0777/nodemain' : 'anton0777/nodedev'}"
        IMAGE_TAG = "v1.0"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Lint Dockerfile') {
            steps {
                sh 'docker run --rm -i hadolint/hadolint < Dockerfile'
            }
        }

        stage('Build') {
            steps {
                nodejs('Node 7.8.0') {
                    sh 'npm install'
                }
            }
        }

        stage('Test') {
            steps {
                nodejs('Node 7.8.0') {
                    sh 'npm test'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-creds') {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push("${IMAGE_TAG}")
                    }
                }
            }
            post {
                success {
                    script {
                        if (BRANCH_NAME == 'main') {
                            build job: 'Deploy_to_main', wait: false
                        } else if (BRANCH_NAME == 'dev') {
                            build job: 'Deploy_to_dev', wait: false
                        }
                    }
                }
            }
        }
    }
}