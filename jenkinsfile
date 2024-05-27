pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'DockerHub_credentials'
        DOCKER_IMAGE_NAME = 'doravissar/k8s'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from your source control
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    docker.build("${env.DOCKER_IMAGE_NAME}:latest", ".")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', "${env.DOCKER_CREDENTIALS_ID}") {
                        // Push the Docker image to Docker Hub
                        docker.image("${env.DOCKER_IMAGE_NAME}:latest").push()
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
