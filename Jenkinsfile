pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_cred')
        GIT_CREDENTIALS = 'github_cred'
        DOCKER_IMAGE = "doravissar/k8s_deploy"
        VERSION = "${env.BUILD_NUMBER}"
        KUBE_CONFIG = credentials('kubernets_cred')
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
                    withCredentials([file(credentialsId: 'kubernets_cred', variable: 'KUBE_CONFIG_FILE')]) {
                        sh "kubectl --kubeconfig=${KUBE_CONFIG_FILE} config set-credentials jenkins-user --client-key=${KUBE_CONFIG_FILE}"
                        sh "kubectl --kubeconfig=${KUBE_CONFIG_FILE} config set-context --current --user=jenkins-user"
                        sh "kubectl --kubeconfig=${KUBE_CONFIG_FILE} set image deployment/flask-app flask-app=${DOCKER_IMAGE}:${VERSION} --namespace=jenkins --record"
                    }
                }
            }
        }
    }
}