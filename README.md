# Customer Portal – CI/CD Pipeline with Jenkins, Docker & Kubernetes

## Overview

This project demonstrates an automated CI/CD pipeline for a containerized Flask application using Jenkins, Docker, Docker Hub, and Kubernetes.

The pipeline automates the process of building a Docker image, pushing it to Docker Hub, deploying the updated application to a Kubernetes cluster, and verifying its health.

## Architecture

```text
          Developer
              |
              v
         GitHub Repository
              |
              v
            Jenkins
              |
       +------+------+
       |             |
       v             v
   Build Docker   Push Image
      Image       to Docker Hub
                       |
                       v
                Kubernetes Cluster
                       |
                       v
                Update Deployment
                       |
                       v
                 Rollout Status
                       |
                       v
                  Smoke Test
                  /health
                       |
                       v
                 Deployment
                  Successful
```

## Technologies Used

- **Python / Flask** – Application development.
- **Git & GitHub** – Source code management.
- **Jenkins** – CI/CD automation.
- **Docker** – Application containerization.
- **Docker Hub** – Container image registry.
- **Kubernetes** – Container orchestration and deployment.
- **Minikube** – Local Kubernetes cluster.
- **Ubuntu Linux** – Development environment.

## Application Endpoints

| Endpoint | Description |
|---|---|
| `/` | Returns application information in JSON format. |
| `/health` | Health check endpoint used to verify application availability. |

## CI/CD Pipeline Workflow

The Jenkins pipeline consists of the following stages:

1. **Checkout:** Retrieves the latest source code and Jenkinsfile from the GitHub repository.
2. **Build Image:** Builds a Docker image using the application Dockerfile.
3. **Push Image:** Authenticates with Docker Hub and pushes the image using the Git commit hash as its tag.
4. **Deploy to Kubernetes:**
   - Updates the Kubernetes Deployment with the new image.
   - Waits for the rollout to complete.
   - Performs a health check against the application using the `/health` endpoint.
5. **Rollback:** If deployment verification fails, Jenkins attempts to roll back to the previous deployment revision.

## Kubernetes Configuration

- **Namespace:** `dev`
- **Deployment:** `customer-portal`
- **Service:** `customer-portal`
- **Replicas:** 2
- **Container Port:** 5000
- **Deployment Authentication:** Kubernetes ServiceAccount with namespace-scoped RBAC permissions.

## Prerequisites

Before running the project, make sure you have:

- Git
- Docker
- Jenkins with Docker CLI access
- Kubernetes cluster (Minikube)
- kubectl
- Docker Hub account
- Jenkins credentials configured for Docker Hub and Kubernetes access

## Jenkins Credentials

Configure the following credentials in Jenkins:

| Credential ID | Type | Purpose |
|---|---|---|
| `dockerhub-credentials` | Username with password | Docker Hub authentication |
| `kubeconfig-dev` | Secret file | Kubernetes cluster authentication |

**Note:** Credentials and access tokens are stored in Jenkins Credentials and are not hardcoded in the Jenkinsfile.

## Running the Application Locally

Clone the repository:

```bash
git clone https://github.com/mosubehi/customer-portal.git
cd customer-portal
```

Build the Docker image:

```bash
docker build -t customer-portal:local .
```

Run the container:

```bash
docker run -d -p 5000:5000 --name customer-portal customer-portal:local
```

Test the application:

```bash
curl http://localhost:5000/health
```

Expected response:

```json
{"status": "healthy"}
```

## Monitoring the Kubernetes Deployment

Check deployment status:

```bash
kubectl get deployments -n dev
```

Check running pods:

```bash
kubectl get pods -n dev
```

Check services:

```bash
kubectl get services -n dev
```

View deployment rollout status:

```bash
kubectl rollout status deployment/customer-portal -n dev
```

## Key Features

- Automated CI/CD workflow.
- Docker image versioning using Git commit hashes.
- Automated image publishing to Docker Hub.
- Kubernetes rolling updates.
- Deployment rollout verification.
- Application health checks.
- Automatic rollback on deployment verification failure.
- Secure credential handling through Jenkins Credentials.
- Kubernetes RBAC for deployment permissions.

## Future Improvements

- Integrate automated unit and integration tests.
- Add code quality and security scanning.
- Improve Docker image size and build efficiency.
- Implement a more durable Kubernetes authentication mechanism.
- Add monitoring and logging using Prometheus and Grafana.
- Introduce Helm for Kubernetes deployment management.

## Author

**Mohammed Okleh**

GitHub: [mosubehi](https://github.com/mosubehi)

---

*This project was built as a hands-on DevOps learning project to practice CI/CD automation, containerization, and Kubernetes deployments.*
