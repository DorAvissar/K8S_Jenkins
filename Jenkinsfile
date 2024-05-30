pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_cred')
        GIT_CREDENTIALS = credentials('github_cred')
        DOCKER_IMAGE = "doravissar/k8s_deploy"
        VERSION = "${env.BUILD_NUMBER}"
        REPO_URL = 'https://github.com/DorAvissar/K8S_Jenkins.git'
        BRANCH = 'main'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}", credentialsId: "${GIT_CREDENTIALS}"
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
        
         stage('Update Kubernetes Manifests') {
            steps {
                script {
                    sh '''
                        sed -i 's|image: doravissar/k8s_deploy.*|image: doravissar/k8s_deploy:latest|' deployment.yaml
                        git add deployment.yaml
                        git commit -m "Update deployment.yaml with latest Docker image tag"
                        git push origin main
                    '''
                }
            }
        }
    }
}
