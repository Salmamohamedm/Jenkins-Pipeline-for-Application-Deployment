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
        OPENSHIFT_NAMESPACE = 'salmamakram'
        GITHUB_REPO = "Salmamohamedm/MultiCloudDevOpsProject"
        OPENSHIFT_SERVER = 'https://api.ocpuat.devopsconsulting.org:6443'
        APP_SERVICE_NAME = 'springboot1'
        APP_PORT = '8080'
        APP_HOST_NAME = 'springboot1.apps.ocpuat.devopsconsulting.org'
        
    }
```
  
# Stages of a Jenkins pipeline script
1. Checkout:
     - Clones the source code from the specified GitHub repository.
2. Build Docker image and push to Docker Hub:
      - Builds the Docker image for the application and pushes it to Docker Hub.
      - Utilizes Docker Hub credentials stored in Jenkins for authentication.
      - Cleans up temporary Docker images after build and push operations.
3. Deploy to OpenShift:
      - Authenticates with OpenShift using the provided token.
      - Sets the OpenShift project.
      - Deploys the Docker image to OpenShift.
      - Exposes the service using oc create service and oc expose service commands.
```
    stages {
        
        
        stage('Checkout') {
            steps {
                git url: "https://github.com/${GITHUB_REPO}.git", branch: 'main'
            }
        }
        
        
        stage('Build Docker image and push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_REGISTRY_USERNAME', passwordVariable: 'DOCKER_REGISTRY_PASSWORD')]) {
                    script {
                        try {
                            sh "echo \${DOCKER_REGISTRY_PASSWORD} | docker login -u \${DOCKER_REGISTRY_USERNAME} --password-stdin"
                            sh "docker build -t ${imageNameapp} ."
                            sh "docker tag ${imageNameapp} ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${imageTagApp}"
                            sh "docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${imageTagApp}"
                        } finally {
                            // Clean up even if the build or push fails
                            sh "docker rmi -f ${imageNameapp}"
                        }
                    }
                }
            }
        }


        

        stage('Deploy to OpenShift') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'openshift-token', variable: 'OPENSHIFT_SECRET')]) {
                    sh "oc login --token=\${OPENSHIFT_SECRET} --server=\${OPENSHIFT_SERVER} --insecure-skip-tls-verify"
                    }
                    sh "oc project \${OPENSHIFT_PROJECT}"
                    //sh "oc delete dc,svc,deploy,ingress,route \${DOCKER_IMAGE} || true"
                    sh "oc new-app ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${imageTagApp} --token=\${OPENSHIFT_SECRET}"
                    
                    // Expose the service 
                    sh "oc create service clusterip ${APP_SERVICE_NAME} --tcp=8080:8080 "
                    sh " oc expose service/${APP_SERVICE_NAME}"
                    
                    //sh "oc create route edge --service \${APP_SERVICE_NAME} --port \${APP_PORT} --hostname springboot.apps.ocpuat.devopsconsulting.org --insecure-policy Redirect"

                }
            }
        }
    }
}

```
