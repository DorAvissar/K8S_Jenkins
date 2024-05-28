pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred')
        GIT_CREDENTIALS = 'github_cred'
        DOCKER_IMAGE = "DorAv/k8s_deploy"
        VERSION = "${env.BUILD_NUMBER}"
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
                    docker.withRegistry('', 'dockerhub-credentials') {
                        dockerImage.push()
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            environment {
                KUBECONFIG = credentials('kubernets_cred')
            }
            steps {
                script {
                    def deploymentName = "k8s-deployment"
                    def containerName = "flask-app"
                    def image = "${DOCKER_IMAGE}:${VERSION}"
                    def namespace = "jenkins"

                    sh "kubectl set image deployment/${deploymentName} ${containerName}=${image} -n ${namespace} --record"
                }
            }
        }
    }
}