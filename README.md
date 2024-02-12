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
      
