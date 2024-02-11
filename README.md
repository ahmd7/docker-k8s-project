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
- [Monitoring](#monitoring)
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
## Contributing

Pull requests are welcome. For major changes, please open an issue first
to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License

[MIT](https://choosealicense.com/licenses/mit/)