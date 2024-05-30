# k8s-project
This project showcases how Kubernetes, Jenkins, DockerHub, ArgoCD, and GitHub work together. Its aim is to automate the deployment process of a basic Flask application using CI/CD. Whenever modifications are made to the app.py file on GitHub, Jenkins constructs a Docker image, uploads it to DockerHub, and then modifies the deployment.yaml file. This triggers ArgoCD to deploy the updated application to the Kubernetes cluster.

  ![get-context](https://github.com/DorAvissar/K8S_Jenkins/blob/main/Diagram.png?raw=true)
  
## Plugins Used
- **Docker Plugin**
- **Docker Commons Plugin**
- **Docker-build-step**
- **Git Plugin**
- **GitHub Plugin**


## Project Workflow

1. **Setup Kubernetes Cluster**
    - Created a Kubernetes cluster using Kind and configured it with `cluster-config.yaml`.
    
    ```sh
    kind create cluster --config cluster-config.yaml
    kind get clusters
    kubectl create namespace jenkins
    kubectl config set-context --current --namespace=jenkins (to connect the namespace to the cluster)
    kubectl config get-contexts (to verify the cluster)
    ```
    ![get-context](https://github.com/DorAvissar/K8S_Jenkins/assets/165499842/706ccf33-c77e-4936-b4da-9befbbbf3845)

2. **Run Jenkins Container**
    - Started Jenkins container using image created by the docker file (attached in the repo).
    - I created the image of jenkins that support docker commands to use them on the pipeline (docker build . -t <imagename>). 
    - run the following commands: 
    
    ```sh
    docker run -p 9090:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock <imagename> 
    docker exec -it -u root <container name> /bin/bash
    chown root:docker /var/run/docker.sock
    ```
    - now i was able to run jenkins locally on my local host and access it via the web server

3. **Configure ArgoCD**
    - This will create a new namespace, ArgoCD, where Argo CD services and application resources will live.
    ```sh
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

    - how to get the url of the ArgoCD? 
    ```sh
    kubectl get svc -n argocd
    kubectl port-forward svc/argocd-server -n argocd 9090:443
    ```
![argoserver](https://github.com/DorAvissar/K8S_Jenkins/assets/165499842/5f7e375a-2197-4f2e-a2b8-0cc833d38ad4)

    - ArgoCD username : admin
    - ArgoCD password : argocd admin initial-password -n argocd

4. **Connect Jenkins to GitHub**
    - Set up a GitHub webhook and configured Jenkins credentials to trigger the pipeline upon a commit 
    - Used Ngrok to expose Jenkins to the public for GitHub webhook integration.
    - **Note that each time Ngrok is restarted, the URL changes, requiring updating the webhook in GitHub accordingly.**

5. **Connect Jenkins to DockerHub**
    - create Docker Hub credentials in Jenkins using a username and password (make sure that the password is TOKEN , and now your github password)
![cred](https://github.com/DorAvissar/K8S_Jenkins/assets/165499842/9e4be6e8-dd64-4046-86d5-64fa53b933c6)


6. **Configure Jenkins Pipeline [jenkinsfile]**
    - Checkout:
        - Purpose: This stage ensures that the latest code from the specified branch in the GitHub repository is checked out for further processing.
        - Steps:
        - It retrieves the author's name of the last commit made to the repository.
        - If the author is identified as "Jenkins", it skips the build to avoid infinite loops where Jenkins continuously builds itself.
        - It then proceeds to checkout the code from the specified branch in the GitHub repository using the provided credentials.

    - Build Docker Image:
        - Purpose: This stage builds a Docker image of the Flask application using the Dockerfile present in the repository.
        - Steps:
        - It utilizes Docker's build functionality to construct the Docker image with the specified tag.
    
    - Push to DockerHub:
        - Purpose: This stage pushes the built Docker image to DockerHub for storage and distribution.
        - Steps:
        - It uses Docker's registry functionality along with DockerHub credentials to authenticate and push the Docker image.

    - Update Kubernetes Deployment.yaml:
        - Purpose: This stage updates the Kubernetes deployment manifest file (deployment.yaml) with the latest version of the Docker image.
        - Steps:
        - It employs a sed command to replace the existing image tag in the deployment.yaml file with the newly built Docker image's tag.
        - Then, it commits this change to the GitHub repository, using Jenkins' credentials to authenticate, along with the provided username and password.
        - Finally, it pushes the updated deployment.yaml file to the GitHub repository's main branch.

7. **Resolve Deployment Issues**
    - Encountered `kubectl: not found` error, resolved by installing kubectl inside the Jenkins container.
    
    ```sh
    apt-get update && apt-get install -y apt-transport-https gnupg2 curl
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    ```
    - **Note that for the deployment to work, you need to manually deploy the application for the first time**

8. **Deployment Verification**
    - Verified the deployment using the ArgoCD interface, ensuring new pods replace the old ones.

    - **So ,How can I access the Flask application after the deployment?** 
    ```sh
    kubectl port-forward pod/<pod name> 8080:80 -n <name space>
    ```
    ![getapp](https://github.com/DorAvissar/K8S_Jenkins/assets/165499842/a2d85685-499e-4194-885f-f663733ed1c7)
   
    - run http://localhost:8080
    
    ![app](https://github.com/DorAvissar/K8S_Jenkins/assets/165499842/90dd980e-2b14-4277-9d9f-31081232f255)


## Troubleshooting

### Docker Commands Not Found
    - Issue: Jenkins container did not recognize Docker commands.
    - Solution: started the Docker daemon manually.

### kubectl Commands Not Found
    - Issue: Jenkins container did not recognize `kubectl` commands.
    - Solution: Installed kubectl manually and configured kubeconfig.

### GitHub Webhook and Localhost Issues
    - Issue: Webhook did not work due to Jenkins running on localhost.
    - Solution: Used Ngrok to expose Jenkins to the public and allow GitHub to trigger the webhook.

## Summary
The pipeline workflow begins by checking out the latest code from a GitHub repository. It then builds a Docker image of a Flask application, pushes it to DockerHub, and updates the Kubernetes deployment manifest file with the new image version. Finally, it commits and pushes this change back to the GitHub repository.
The outcome of the pipeline is an automated deployment of the Flask application on a Kubernetes cluster. Whenever changes are made to the app.py file and pushed to GitHub, Jenkins automatically triggers the pipeline, ensuring that the latest version of the application is built, deployed, and updated in the Kubernetes cluster without manual intervention.

Following the pipeline's completion, ArgoCD takes over, detecting changes in the Git repository's Kubernetes manifests. It then synchronizes the live state of the application with the desired state specified in the Git repository. This ensures that any updates pushed through the pipeline are automatically deployed to the Kubernetes cluster, maintaining consistency between the defined configuration in Git and the actual running application.

![argocd](https://github.com/DorAvissar/K8S_Jenkins/assets/165499842/6e2adbc8-f994-4c8c-ad57-ecdcdb8777ac)


