# Jenkins-Pipeline-for-Application-Deployment
This Jenkins pipeline automates the build, Dockerization, pushing to Docker Hub, pulling from Docker Hub, and deployment of an application on an OpenShift Cluster within your namespace.
# Objective
The objective of this pipeline is to streamline the deployment process of your Java web application onto an OpenShift Cluster, ensuring consistency, reliability, and efficiency.
# Prerequisites
Before running this pipeline, ensure you have the following prerequisites:
- Jenkins installed and configured with necessary plugins
- Docker Hub account with appropriate credentials configured in Jenkins
- OpenShift Cluster access with credentials configured in Jenkins
- Maven or Gradle for building the application
- Docker installed on the Jenkins server
# Pipeline Steps
1. Build: Builds the application using Maven or Gradle.
2. Dockerize: Creates a Docker image for the application.
3. Push to Docker Hub: Pushes the Docker image to a Docker Hub repository.
4. Pull from Docker Hub: Pulls the Docker image from Docker Hub.
5. Deploy to OpenShift: Deploys the application on an OpenShift Cluster within your namespace.
      
# Usage
- To utilize this Jenkins pipeline, follow these instructions:
1. Begin by creating a new Jenkins pipeline job.
2. Proceed to copy the provided pipeline script (Jenkinsfile) into the pipeline configuration.
3. Configure the required environment variables, including credentials, Docker image name, application name, and OpenShift namespace.
4. Initiate the pipeline either manually or by setting up automatic triggering based on changes in your code repository.

# Environment Variables
- DOCKER_HUB_CREDENTIALS: Jenkins credentials ID for Docker Hub authentication.
- OPENSHIFT_CREDENTIALS: Jenkins credentials ID for OpenShift Cluster authentication.
- DOCKER_REGISTRY: The Docker registry where the Docker images will be pushed.
- DOCKER_IMAGE: Name of the Docker image for your application.
- imageTagApp: A unique tag for the Docker image, including the build number.
- imageNameapp: The full name of the Docker image, including the Docker registry and image tag.
- OPENSHIFT_PROJECT: The OpenShift project where the application will be deployed.
- GITHUB_REPO: The GitHub repository containing the application source code.
- OPENSHIFT_SERVER: The URL of the OpenShift server.
- APP_SERVICE_NAME: The name of the OpenShift service associated with the application.
- APP_PORT: The port on which the application will be accessible.
- APP_HOST_NAME: The hostname used to access the application.
- OPENSHIFT_NAMESPACE: Namespace on the OpenShift Cluster where the application will be deployed.

```
environment {
        
        DOCKER_REGISTRY = "salmamakram"
        DOCKER_IMAGE = "spring-boot"
        imageTagApp = "build-${BUILD_NUMBER}-app"
        imageNameapp = "${DOCKER_REGISTRY}:${imageTagApp}"
        OPENSHIFT_PROJECT = 'salmamakram'
        GITHUB_REPO = "Salmamohamedm/MultiCloudDevOpsProject"
        OPENSHIFT_SERVER = 'https://api.ocpuat.devopsconsulting.org:6443'
        APP_SERVICE_NAME = 'springboot1'
        APP_PORT = '8080'
        APP_HOST_NAME = 'springboot1.apps.ocpuat.devopsconsulting.org'
        
    }
```
  
