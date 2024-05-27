pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_cred')
        KUBECONFIG_CREDENTIALS = credentials('k8s_cred')
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
                    dockerImage = docker.build("doravissar/k8s:latest")
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
                    withCredentials([file(credentialsId: 'kind-kubeconfig', variable: 'KUBECONFIG')]) {
                        sh 'kubectl create deployment my-app --image=doravissar/k8s:latest --kubeconfig=$KUBECONFIG'
                        sh 'kubectl expose deployment my-app --port=8080 --type=LoadBalancer --kubeconfig=$KUBECONFIG'
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
