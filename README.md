# Jenkins Pipeline for Application Deployment
This Jenkins pipeline automates the build, Dockerization, pushing to Docker Hub, pulling from Docker Hub, and deployment of an application on an OpenShift Cluster within your namespace.
# Overview
The primary objective of this pipeline is to optimize the deployment workflow for Java web applications onto an OpenShift Cluster. By streamlining this process, the pipeline aims to enhance consistency, reliability, and efficiency throughout the deployment lifecycle. Through automation and standardized procedures, the pipeline seeks to minimize errors, reduce deployment time, and ensure seamless application delivery onto the OpenShift Cluster.
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
# Usage:
To utilize this Jenkins pipeline, follow these instructions:
1. Begin by creating a new Jenkins pipeline job.
2. Proceed to copy the provided pipeline script (Jenkinsfile) into the pipeline configuration.
3. Configure the required environment variables, including credentials, Docker image name, application name, and OpenShift namespace.
4. Initiate the pipeline either manually or by setting up automatic triggering based on changes in your code repository.


> [!NOTE]
By following these steps, you can efficiently automate your application deployment using Jenkins.

# Environment Variables
- DOCKER_REGISTRY: The Docker registry where the Docker images will be pushed.
- DOCKER_IMAGE: The name of the Docker image.
- imageTagApp: A unique tag for the Docker image, including the build number.
- imageNameapp: The full name of the Docker image, including the Docker registry and image tag.
- OPENSHIFT_NAMESPACE: The OpenShift project where the application will be deployed.
- GITHUB_REPO: The GitHub repository containing the application source code.
- OPENSHIFT_SERVER: The URL of the OpenShift server.
- APP_SERVICE_NAME: The name of the OpenShift service associated with the application.
- APP_PORT: The port on which the application will be accessible.
- APP_HOST_NAME: The hostname used to access the application.

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
# Jenkins pipeline script 
This Jenkins pipeline script consists of three stages:
1. Checkout:
     - This stage checks out the source code of the application from the specified GitHub repository.
     - It uses the git step to clone the repository URL and checkout the main branch.
2. Build Docker image and push to Docker Hub:
     - This stage builds the Docker image for the application and pushes it to Docker Hub.
     - It uses the withCredentials block to securely access Docker Hub credentials stored in Jenkins.
       
- Inside the script block:
  
     - It logs in to Docker Hub using the provided credentials.
     - Builds the Docker image using the docker build command with the specified image name.
     -  Tags the built image with the full image name including the Docker registry, image name, and tag.
     - Pushes the tagged image to Docker Hub repository.
     - Finally, it cleans up by removing the temporary Docker image built during the process, even if the build or push fails.
3. Deploy to OpenShift:
     - This stage deploys the application to an OpenShift Cluster.
- Inside the script block:
     - It uses OpenShift credentials stored in Jenkins to authenticate with the OpenShift Cluster.
     - Sets the project to the specified OpenShift project.
     - Deploys the Docker image to OpenShift using the oc new-app command.
     - Exposes the service using 'oc' create service and 'oc' expose service commands.
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
![Screenshot (453)](https://github.com/Salmamohamedm/Jenkins-Pipeline-for-Application-Deployment/assets/109488469/19fced9f-26a4-4e78-86d0-d472fde1e830)



> [!TIP]
> You can further customize and extend these stages as per your project requirements, such as adding additional steps for testing, monitoring, or integrating with other systems.
       
# Contributing 
Feel free to contribute to this Jenkins pipeline by suggesting improvements or submitting pull requests.
