# k8s-project
This project demonstrates the integration between Kubernetes, Jenkins, DockerHub, and GitHub. The goal is to automate the deployment of a simple Flask application using a CI/CD pipeline. When changes are pushed to the `app.py` file on GitHub, Jenkins builds a Docker image, pushes it to DockerHub, and deploys it to a Kubernetes cluster.

## Plugins Used
- **Kubernetes CLI Plugin**
- **Kubernetes Continuous Deploy Plugin**
- **Kubernetes Credentials Plugin**
- **Kubernetes Plugin**
- **Docker Plugin**
- **Docker Commons Plugin**
- **Docker-build-step**
- **Git Plugin**
- **GitHub Plugin**


## Project Workflow
1. **Setup Kubernetes Cluster**
    - Created a Kubernetes cluster using Kind and configured it with `jenkins-config.yaml`.
    
    ```sh
    kind create cluster --config jenkins-config.yaml
    kind get clusters
    kubectl create namespace jenkins
    kubectl config set-context --current --namespace=jenkins (to connect the namespace to the cluster)
    kubectl config get-contexts (to verify the cluster)
    ```
    
2. **Run Jenkins Container**
    - Started Jenkins container using image created by the docker file (attached in the repo).
    - I created the image of jenkins that support docker (docker build . -t <imagename>). 
    - run the following commands: 
    
    ```sh
    docker run -p 9090:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock <imagename> 
    docker exec -it -u root <container name> /bin/bash
    chown root:docker /var/run/docker.sock
    ```

    - now i was able to run jenkins locally on my local host and access it via the web server


3. **Integrate Jenkins with Kubernetes**
    - Connected Jenkins to Kubernetes by configuring the Kubernetes cloud settings and using the kubeconfig credentials defined within Jenkins.
    - **The kubeconfig file details are located in the .kube directory at the following path: C:/Users/Username/.kube**
    - **find the url by doing the following command "kubectl cluster-info --context kind-k8s-jenkins-cluster"**

4. **Connect Jenkins to GitHub**
    - Set up a GitHub webhook and configured Jenkins credentials to trigger the pipeline upon a commit 
    - Used Ngrok to expose Jenkins to the public for GitHub webhook integration.
    - **Note that each time Ngrok is restarted, the URL changes, requiring updating the webhook in GitHub accordingly.**


5. **Connect Jenkins to DockerHub**
    - create Docker Hub credentials in Jenkins using a username and password


6. **Configure Jenkins Pipeline [jenkinsfile]**
    - agent any: This specifies that the pipeline can run on any available Jenkins agent.
    environment: Defines environment variables for Docker Hub credentials, Git credentials, Docker image name, and the build version.

    - stage('Checkout'): Checks out the code from the main branch of the specified GitHub repository using the provided Git credentials.

    - stage('Build Docker Image'): Builds a Docker image from the checked-out code, tagging it with the version number based on the Jenkins build number. 

    - stage('Push to DockerHub'): Authenticates with Docker Hub using the provided credentials and pushes the built Docker image to the Docker Hub repository.

    - stage('Deploy to Kubernetes'): Deploys the new Docker image to a Kubernetes cluster.
    environment: Sets the Kubernetes configuration using the provided credentials.
    script: Defines deployment details and updates the Kubernetes deployment with the new Docker image version, recording the change.

7. **Resolve Deployment Issues**
    - Encountered `kubectl: not found` error, resolved by installing kubectl inside the Jenkins container.
    
    ```sh
    apt-get update && apt-get install -y apt-transport-https gnupg2 curl
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    ```
    - **Note that for the deployment to work, you need to manually deploy the application for the first time**

8. **Deployment Verification**
    - Verified the deployment using Lens, ensuring new pods replace the old ones.

## Troubleshooting

### Docker Commands Not Found
    - Issue: Jenkins container did not recognize Docker commands.
    - Solution: Installed Docker Inside the Jenkins container and started the Docker daemon manually.

### kubectl Commands Not Found
    - Issue: Jenkins container did not recognize `kubectl` commands.
    - Solution: Installed kubectl manually and configured kubeconfig.

### GitHub Webhook and Localhost Issues
    - Issue: Webhook did not work due to Jenkins running on localhost.
    - Solution: Used Ngrok to expose Jenkins to the public and allow GitHub to trigger the webhook.

## Summary
In summary, this Jenkinsfile defines a declarative pipeline with three stages: building a Docker image, pushing the image to Docker Hub, and deploying the image to a Kubernetes cluster. The pipeline is designed to run on any available agent and includes logic to fetch code from a GitHub repository, build and push the Docker image to Docker Hub, and finally deploy it to Kubernetes.