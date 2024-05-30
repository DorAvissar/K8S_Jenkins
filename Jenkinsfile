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
                    // Ensure git is installed in the Jenkins container using appropriate package manager
                    sh '''
                        if command -v apt-get &> /dev/null; then
                            apt-get update && apt-get install -y git
                        elif command -v yum &> /dev/null; then
                            yum install -y git
                        elif command -v apk &> /dev/null; then
                            apk add git
                        else
                            echo "No supported package manager found for installing git."
                            exit 1
                        fi
                    '''
                    
                    // Update deployment.yaml file with the new image version
                    sh """
                        sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${VERSION}|' deployment.yaml
                    """
                    
                    // Configure git and commit the changes
                    sh """
                        git config --global user.email "jenkins@yourdomain.com"
                        git config --global user.name "Jenkins"
                        git add deployment.yaml
                        git commit -m "Update image to ${DOCKER_IMAGE}:${VERSION}"
                        git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/DorAvissar/K8S_Jenkins.git ${BRANCH}
                    """
                }
            }
        }
    }
}
