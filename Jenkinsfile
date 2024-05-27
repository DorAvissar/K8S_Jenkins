pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_cred')
        KUBECONFIG_CREDENTIALS = credentials('k8s_cred')
        VERSION = 'latest' // Define the VERSION variable here

    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DorAvissar/K8S_Jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image and capture the image name
                    dockerImage = docker.build("doravissar/k8s:latest")
                    // Store the image name in a variable
                    DOCKER_IMAGE = "doravissar/k8s:latest"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub_cred') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'k8s_cred', variable: 'KUBECONFIG')]) {
                        // Use the stored image name variable
                        sh "kubectl --kubeconfig=config set image deployment/flask-app flask-app=${DOCKER_IMAGE}:${VERSION} --record"
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
