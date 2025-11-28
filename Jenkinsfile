pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'faizann16'
        BACKEND_IMAGE = "${DOCKERHUB_USER}/mean-backend"
        FRONTEND_IMAGE = "${DOCKERHUB_USER}/mean-frontend"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test Backend') {
            steps {
                dir('backend') {
                    sh 'npm install'
                    // optional: sh 'npm test'
                }
            }
        }

        stage('Build & Test Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    // optional: sh 'npm test'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh """
                      docker build -t ${BACKEND_IMAGE}:${BUILD_NUMBER} -t ${BACKEND_IMAGE}:latest backend
                      docker build -t ${FRONTEND_IMAGE}:${BUILD_NUMBER} -t ${FRONTEND_IMAGE}:latest frontend
                    """
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                     usernameVariable: 'DOCKERHUB_USERNAME',
                                                     passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh """
                          echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
                          docker push ${BACKEND_IMAGE}:${BUILD_NUMBER}
                          docker push ${BACKEND_IMAGE}:latest
                          docker push ${FRONTEND_IMAGE}:${BUILD_NUMBER}
                          docker push ${FRONTEND_IMAGE}:latest
                          docker logout
                        """
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                script {
                    // docker-compose.yml is in workspace root
                    sh """
                      docker compose pull
                      docker compose up -d --remove-orphans
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Build/Deployment failed.'
        }
    }
}
