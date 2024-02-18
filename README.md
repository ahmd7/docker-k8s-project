# Docker-K8s-project-on-ECS

This repository contains the source code and deployment configuration for a 3-tier application built using Docker and orchestrated with Kubernetes, deployed on Amazon ECS (Elastic Container Service).

This repo will focus on the deployment of the application not the code itself.
## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Frontend docker-file](#Frontend-docker-file)
- [Backend docker-file](#Backend-docker-file)
- [Kubernetes manifests](#kubernetes-manifests)
- [Deployment](#deployment)
- [Contributing](#contributing)
- [License](#license)

## Overview

This 3-tier application is composed of three main components:

1. **Frontend**: The user interface layer built with [Node.js](https://nodejs.org/).
2. **Backend**: The application logic layer implemented using [Node.js](https://nodejs.org/).
3. **Database**: The data storage layer powered by [MongoDB](https://www.mongodb.com/).

The application is containerized using Docker, and the orchestration is managed with Kubernetes, specifically deployed on Amazon ECS for scalability and flexibility.
## Prerequisites

Before getting started, ensure you have the following tools installed:

- [Docker](https://www.docker.com/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [AWS CLI](https://aws.amazon.com/cli/)
- [eksctl](https://eksctl.io/)

## Project Structure
![Alt 3-tier](/readme-images/3-tier.png)

**frontend/:** Contains the frontend application code and Dockerfile.

**backend/:** Contains the backend application code , the MongoDB database configuration and Dockerfile.

**k8s/:** Kubernetes deployment files for each tier.

## Frontend docker-file

This Dockerfile is used to build a Docker image for a **Node.js** application. Let's break down each instruction: 
```dockerfile
FROM node:18

WORKDIR /app

COPY package*.json .

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "run", "start"]
```
**FROM node:18:** Uses Node.js 18 as the base image.

**WORKDIR /app:** Sets the working directory inside the container to /app.

**COPY package.json . :** Copies package files for efficient dependency installation.

**RUN npm install:** Installs Node.js application dependencies.

**COPY . .:** Copies the application code into the container.

**EXPOSE 3000:** Documents that the application uses port 3000.

**CMD ["npm", "run", "start"]:** Sets the default command to start the Node.js application.

## Backend docker-file
This Dockerfile is used to build a Docker image for a **Node.js** application. Which is same as the frontend:

```dockerfile
FROM node:18

WORKDIR /app

COPY package*.json .

RUN npm install

COPY . .

EXPOSE 8080

CMD [ "node", "RUN" , "index.js" ]

```
## Kubernetes manifests

This section includes Kubernetes manifest files for deploying and managing the application in a Kubernetes cluster.
 1. **`k8s_manifests/backend-deployment.yaml`**
This Kubernetes Deployment YAML file defines the deployment configuration for a backend application. It orchestrates the deployment of a single replica of a containerized backend service within the workshop namespace. The deployment follows a Rolling Update strategy with specific surge and unavailability settings. The container runs an image from a specified container registry, with environment variables set for MongoDB connection details, including sensitive information retrieved from a Kubernetes Secret (mongo-sec). The container exposes port 8080 and is configured with liveness and readiness probes to ensure the health and readiness of the application.
 
 2. **`k8s_manifests/backend-service.yaml`**
backend within the workshop namespace. The service exposes port 8080 using the ClusterIP type, directing traffic to pods labeled with role: backend. It facilitates internal communication among backend pods within the cluster.

 3. **`k8s_manifests/frontend-deployment.yaml`**
This Kubernetes Deployment YAML file configures the deployment of a frontend application within the workshop namespace. It orchestrates a single replica of a containerized frontend service using a Rolling Update strategy. The container runs an image from a specified container registry, with an environment variable set to the backend service's URL. The frontend container exposes port 3000, and the deployment ensures that the application is highly available, with surge and unavailability settings specified for the update strategy.
 
 4. **`k8s_manifests/frontend-service.yaml`**
This Kubernetes Service YAML file defines a service named frontend within the workshop namespace. The service exposes port 3000 using the ClusterIP type, directing traffic to pods labeled with role: frontend. It facilitates internal communication among frontend pods within the cluster.
 5. **`k8s_manifests/full_stack_lb.yaml`**
This Kubernetes Ingress YAML file configures an Application Load Balancer (ALB) named mainlb within the workshop namespace. It defines rules for routing external HTTP traffic to services based on specified paths:

 Requests to /backend are directed to the backend service on port 8080.
Requests to the root path / are directed to the frontend service on port 3000.
 Annotations provide additional configuration details, specifying the ALB's scheme, target type, and listen ports. The Ingress is associated with the alb Ingress class. This configuration enables external access and load balancing for the backend and frontend services.
 1. **`k8s_manifests/mongo/deploy.yaml`**
  This Kubernetes Deployment YAML file configures the deployment of a MongoDB instance within the workshop namespace. It orchestrates a single replica of a containerized MongoDB using the official `mongo:4.4.6` image. Key configurations include:

 **Container Configuration:**

  - Image: `MongoDB version 4.4.6.`
Command: Utilizes numactl for memory management and sets MongoDB parameters.
Ports: Exposes port 27017 for MongoDB connections.
Resource Limits: Specifies resource requests and limits for memory and CPU usage.
Environment Variables: Retrieves MongoDB root username and password from a Kubernetes Secret (`mongo-sec`).
Replicas: Ensures a single MongoDB replica.

  - Storage Configuration (commented out): Volume-related configurations (persistent volume claims and mounts) are currently commented out. Uncommenting these sections would enable persistent storage for MongoDB data.

  - This Deployment is designed for MongoDB deployment with resource constraints and can be applied to a Kubernetes cluster using
```bash
kubectl apply -f mongodb-deployment.yaml.
```

1. **`k8s_manifests/mongo/secrets.yaml`**
  This Kubernetes Secret YAML file defines a secret named mongo-sec within the workshop namespace. It is of type Opaque, indicating generic, arbitrary data. This secret contains sensitive information, specifically:

 **Username:** The Base64-encoded value for 'admin'.
**Password:** The Base64-encoded value for 'password123'.
This secret is intended for use with other Kubernetes resources, such as Deployments, allowing secure storage and retrieval of sensitive data, in this case, MongoDB root username and password. The values can be decoded from Base64 when needed.
1. **`k8s_manifests/mongo/service.yaml`**
This Kubernetes Service YAML file defines a service named mongodb-svc within the workshop namespace. It exposes a MongoDB instance to other services within the cluster. Key configurations include:

 **Selector:** Identifies pods to include in the service using the label app: mongodb.
Ports: Maps port 27017 on the service to the same port on the target pods. The service is accessible internally within the cluster on this port.
This service allows other components in the same namespace to connect to the MongoDB instance using the service name (`mongodb-svc`) and port (`27017`). It facilitates communication between different parts of the application stack.
## Deployment
This section outlines the deployment process for the application, covering distinct phases from the initial setup of infrastructure components, such as EC2 instances and IAM users, to building frontend and backend images, Kubernetes deployment, setting up an Application Load Balancer (ALB) with Ingress, and concluding with a streamlined cleanup process in Phase 5. Each phase is designed to contribute to the overall deployment and management of the application stack.

### 1. AWS setup
- You should create a user in AWS with CLI access from the AWS Management Console by following these steps. Begin by navigating to the **IAM** (Identity and Access Management) service in the AWS Console. Choose "Users" from the left-hand menu, and then click on the "Add user" button. Specify a username, select **Programmatic access** for CLI access, and proceed to set permissions. You should  attach existing policies that grant necessary permissions. After configuring permissions. Finally, click "Create user." AWS will provide you with access key credentials that you should securely store.
- create an EC2 instance on AWS from the console and connect to it via SSH, start by navigating to the EC2 service. Once the instance is running, select it from the EC2 dashboard, and click "Connect." Follow the provided SSH connection command to connect to the instance. Once connected, make a directory using the `mkdir` command and clone the repository using in the newly created directory.
```bash
git clone https://github.com/ahmd7/docker-k8s-project.git
```
- set up the AWS CLI, a tool facilitating interaction with AWS services through commands, follow these steps:
  
  1. Install AWS CLI using the following commands:
    ```bash
    apt install aws-cli --classic
    ```
  2. Configure AWS CLI by running:
    ```bash
    aws configure
    ```
    You will be prompted for access and secret keys; retrieve them from the CSV file downloaded earlier.

  3. Keep all other configurations unchanged and press enter. Your AWS CLI is now set up.

    Next, proceed to set up Docker:


    Install Docker with the following commands:
    
    ```bash
    apt install docker.io
    usermod -aG docker $USER  # Replace with your username (e.g., 'ubuntu')
    newgrp docker
    sudo chmod 777 /var/run/docker.sock
    which docker
    ```
  Finally, setup kubectl:
  1. Set up kubectl, a command-line tool for managing Kubernetes clusters:
  ```bash
  apt install kubectl --classic
  ```
  2. Set up eksctl, a command-line tool for managing Amazon EKS clusters:
  ```bash
  curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
  sudo mv /tmp/eksctl /usr/local/bin
  eksctl version
  ```
-  Build Frontend and Backend Images

    #### Step 1: Setup Elastic Container Registry (ECR)

   1. In the AWS console, search for ECR and create repositories for both frontend and backend. Set visibility to public.

    #### Step 2: Setup Frontend

    1. Navigate to the frontend directory in the terminal and run `ls` to ensure you are in the correct directory.

    2. Go to your ECR repo and click on "View push commands."

    3. Run the following commands in your terminal to build the frontend image and push it to the ECR repository:

   ```bash
   aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/l0l7e4u1
   docker build -t 3-tier-frontend .
   docker tag 3-tier-frontend:latest public.ecr.aws/l0l7e4u1/3-tier-frontend:latest
   docker push public.ecr.aws/l0l7e4u1/3-tier-frontend:latest
   ```
   
   Run a container from the image:

    ```bash
    docker images  # copy the image name from the list
    docker run -d -p 3000:3000 <image-name>:latest
    Access your application at public-ip:3000.
    ```
    #### Step 3: Setup Backend
    
    Navigate to the backend directory in the terminal.

    Go to your ECR repo and click on "View push commands" for the backend repo.

    Run the following commands in your terminal to build the backend image and push it to the ECR repository:

```bash
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/l0l7e4u1
docker build -t 3-tier-backend .
docker tag 3-tier-backend:latest public.ecr.aws/l0l7e4u1/3-tier-backend:latest
docker push public.ecr.aws/l0l7e4u1/3-tier-backend:latest
```
Your backend image is now successfully built and pushed to the Elastic Container Registry, which will be utilized when setting up the Elastic Kubernetes Service.






## Contributing

Pull requests are welcome. For major changes, please open an issue first
to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License

[MIT](https://choosealicense.com/licenses/mit/)
