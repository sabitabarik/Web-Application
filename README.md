**Deployment Steps for the Web Application:**

1. Create and Prepare EC2 Instance
Action: Create an Ubuntu EC2 instance on AWS.
Access: Connect to the server using a terminal (e.g., MobaxTerm).

2. Install Necessary Software
Update all packages (sudo apt update && sudo apt upgrade).
Install the following tools using apt or appropriate package managers:
Docker: For containerization.
Jenkins: For CI/CD orchestration.
Maven: For Java project builds.
Git: For source code management.
Terraform: For infrastructure as code.

3. Set Up EKS Cluster
Use Terraform to provision an Elastic Kubernetes Service (EKS) cluster in AWS.
Configure kubectl to connect the server to the EKS cluster using:

aws eks --region ap-south-1 update-kubeconfig --name devopsshack-cluster

4. Configure Jenkins
Access Jenkins via the browser:
URL: 15.206.73.231:8080.
Install the following plugins:
Docker
Docker Pipeline
Kubernetes
Kubernetes Plugin
Kubernetes CLI
AWS Steps
AWS Credentials

5. Add Credentials
In Jenkins:
Add Docker Hub credentials for pushing images.
Add AWS Credentials for interacting with EKS.
Add Git Credentials for source code access.

6. Define the Jenkins Pipeline
Write a Jenkins pipeline to automate the following CI/CD steps:
Clone the Source Code:
Pull the application code from a Git repository.
Build Docker Image:
Use a Dockerfile to create a Docker image of the application.
Push Docker Image:
Push the built image to Docker Hub (or any container registry).
Deploy Application:
Use Kubernetes manifests to deploy the application to the EKS cluster.
