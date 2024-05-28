pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_cred')
        GIT_CREDENTIALS = 'github_cred'
        DOCKER_IMAGE = "doravissar/k8s_deploy"
        VERSION = "${env.BUILD_NUMBER}"
        KUBE_CONFIG = credentials('kubernets_cred') // Corrected to 'kubernets_cred'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DorAvissar/K8S_Jenkins.git', credentialsId: "${GIT_CREDENTIALS}"
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${VERSION}")
                }
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub_cred') {
                        dockerImage.push()
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'kubernets_cred', variable: 'KUBE_TOKEN')]) {
                        sh "kubectl config set-credentials jenkins-user --token=$KUBE_TOKEN"
                        sh "kubectl config set-context --current --user=jenkins-user"
                        sh "kubectl set image deployment/flask-app flask-app=${DOCKER_IMAGE}:${VERSION} --namespace=jenkins --record"
                    }
                }
            }
        }
    }
}
