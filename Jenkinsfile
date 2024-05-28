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
                        sh "kubectl --kubeconfig=${KUBE_CONFIG_FILE} apply -f - <<EOF\napiVersion: apps/v1\nkind: Deployment\nmetadata:\n  name: k8s-deployment\n  namespace: jenkins\nspec:\n  replicas: 1\n  selector:\n    matchLabels:\n      app: flask-app\n  template:\n    metadata:\n      labels:\n        app: flask-app\n    spec:\n      containers:\n      - name: flask-app\n        image: ${DOCKER_IMAGE}:${VERSION}\n        ports:\n        - containerPort: 80\nEOF"
                    }
                }
            }
        }
    }
}
