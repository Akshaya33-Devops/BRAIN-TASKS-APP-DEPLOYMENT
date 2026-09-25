# BRAIN-TASKS-APP-DEPLOYMENT

## Application Deployment using Docker, AWS ECR, Amazon EKS, CodeBuild, CodePipeline and CloudWatch

A complete DevOps deployment project for a React/Vite application using Docker, Kubernetes and AWS DevOps services.

The application is containerized using Docker and served through NGINX. The Docker image is stored in Amazon ECR and deployed to an Amazon EKS cluster. AWS CodeBuild and AWS CodePipeline automate the CI/CD workflow, while Amazon CloudWatch provides centralized logging and monitoring.

## Project Overview

This project demonstrates an end-to-end application deployment workflow:

```text
GitHub
   |
   v
AWS CodePipeline
   |
   v
AWS CodeBuild
   |
   v
Docker Image
   |
   v
Amazon ECR
   |
   v
Amazon EKS
   |
   v
Kubernetes Service
   |
   v
AWS Load Balancer
   |
   v
End User
```

CloudWatch is used for monitoring and centralized logging.

## Technologies Used

- React / Vite
- Docker
- NGINX
- Kubernetes
- Amazon EKS
- Amazon ECR
- AWS CodeBuild
- AWS CodePipeline
- Amazon CloudWatch
- AWS CLI
- kubectl
- GitHub
- Horizontal Pod Autoscaler (HPA)

## AWS Region

```text
ap-south-1
Asia Pacific (Mumbai)
```

## Application Source

The original application source is maintained separately:

https://github.com/Vennilavanguvi/Brain-Tasks-App

The application is provided as a pre-built React/Vite production distribution.

The deployment repository contains the `dist/` directory instead of building the React application from source.

## Deployment Repository

GitHub Repository:

https://github.com/Akshaya33-Devops/BRAIN-TASKS-APP-DEPLOYMENT

Repository structure:

```text
BRAIN-TASKS-APP-DEPLOYMENT/
│
├── dist/
├── Dockerfile
├── deployment.yaml
├── service.yaml
├── buildspec.yml
├── README.md
└── ...
```

## Dockerization

The application is served using NGINX.

### Dockerfile

```dockerfile
FROM nginx:alpine

COPY dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

The `dist/` directory contains the production React/Vite files.

NGINX serves the application on port `80` inside the container.

## Build Docker Image

The Docker image was created using:

```bash
docker build -t brain-tasks-app:latest .
```

Image:

```text
brain-tasks-app:latest
```

## Run Locally

The application was tested locally using:

```bash
docker run -d --name brain-tasks-app-container -p 3000:80 brain-tasks-app:latest
```

Port mapping:

```text
localhost:3000
      |
      v
Container:80
      |
      v
NGINX
      |
      v
React Application
```

Application URL:

```text
http://localhost:3000
```

## Amazon ECR

The Docker image was stored in Amazon Elastic Container Registry.

ECR Repository:

```text
brain-tasks-app
```

Image:

```text
brain-tasks-app:latest
```

Image flow:

```text
Local Docker Image
        |
        v
Amazon ECR
        |
        v
brain-tasks-app:latest
```

## Amazon EKS

The application was deployed to an Amazon EKS cluster.

Cluster name:

```text
brain-tasks-cluster
```

Region:

```text
ap-south-1
```

The Kubernetes deployment pulls the application Docker image from Amazon ECR.

Architecture:

```text
Amazon EKS Cluster
        |
        v
Kubernetes Deployment
        |
        v
Application Pods
        |
        v
Docker Container
        |
        v
React Application
```

## Kubernetes Deployment

The project uses Kubernetes manifests to manage the application.

### Deployment

`deployment.yaml` is responsible for:

- Using the Docker image from ECR
- Creating application pods
- Maintaining the desired number of replicas
- Restarting failed containers

### Service

`service.yaml` exposes the application using a Kubernetes `LoadBalancer` service.

Traffic flow:

```text
Internet
   |
   v
AWS Load Balancer
   |
   v
Kubernetes Service
   |
   v
Application Pod
   |
   v
React Application
```

## Horizontal Pod Autoscaler

A Horizontal Pod Autoscaler was configured for the application.

```bash
kubectl autoscale deployment brain-tasks-app --cpu-percent=70 --min=2 --max=5
```

Configuration:

```text
Minimum replicas : 2
Maximum replicas : 5
CPU target       : 70%
```

The HPA allows the application to automatically adjust the number of pods based on CPU utilization.

## AWS CodeBuild

AWS CodeBuild automates the Docker image build and ECR push process.

Build stages:

```text
Get Source Code
      |
      v
Build Docker Image
      |
      v
Authenticate with ECR
      |
      v
Push Docker Image to ECR
```

CodeBuild project:

```text
brain-tasks-codebuild
```

The project uses GitHub as the source repository.

## AWS CodePipeline

AWS CodePipeline automates the CI/CD workflow.

Pipeline flow:

```text
GitHub
   |
   v
Source Stage
   |
   v
AWS CodeBuild
   |
   v
Build Docker Image
   |
   v
Push Image to ECR
   |
   v
Deploy to Amazon EKS
```

When changes are pushed to the GitHub repository, the pipeline can trigger the deployment workflow.

## Amazon CloudWatch

Amazon CloudWatch is used for monitoring and centralized logging.

Monitoring flow:

```text
CodeBuild  ---> CloudWatch Logs

CodePipeline ---> Pipeline Execution Information

EKS / Application ---> CloudWatch
```

Container Insights log groups used by the project include:

```text
/aws/containerinsights/brain-tasks-cluster/application

/aws/containerinsights/brain-tasks-cluster/dataplane

/aws/containerinsights/brain-tasks-cluster/host
```

## Useful Kubernetes Commands

Update the kubeconfig:

```bash
aws eks update-kubeconfig --region ap-south-1 --name brain-tasks-cluster
```

Check deployments:

```bash
kubectl get deployment
```

Check pods:

```bash
kubectl get pods -o wide
```

Check services:

```bash
kubectl get service
```

Check HPA:

```bash
kubectl get hpa
```

Check resource usage:

```bash
kubectl top pods
```

Check complete application status:

```bash
kubectl get deployment,service,pods -o wide
```

## Troubleshooting

### EKS Cluster Access

If the local system is not connected to the EKS cluster:

```bash
aws eks update-kubeconfig --region ap-south-1 --name brain-tasks-cluster
```

Then verify:

```bash
kubectl get pods
```

### Pods Not Running

Check deployment and pods:

```bash
kubectl get deployment,pods -o wide
```

### Load Balancer Verification

Check the service:

```bash
kubectl get service
```

The application should have an external Load Balancer address.

### CloudWatch Logs

Check CloudWatch log groups:

```bash
aws logs describe-log-groups --region ap-south-1 --query "logGroups[].logGroupName" --output table
```

Check application log streams:

```bash
aws logs describe-log-streams --log-group-name "/aws/containerinsights/brain-tasks-cluster/application" --region ap-south-1
```

### HPA Verification

Check the HPA:

```bash
kubectl get hpa
```

Check CPU and memory metrics:

```bash
kubectl top pods
```

## Project Completion Status

The following project stages were completed and verified:

- Application source cloned
- Production `dist/` build verified
- Dockerfile created
- Docker image built
- Docker container tested locally
- Application accessed through `localhost:3000`
- Docker image pushed to Amazon ECR
- Amazon EKS cluster created and activated
- Kubernetes Deployment configured
- Kubernetes Service configured
- Application exposed through AWS Load Balancer
- AWS CodeBuild configured
- AWS CodePipeline configured
- CloudWatch Container Insights configured
- Application logs verified
- Horizontal Pod Autoscaler configured
- Final application deployment verified

## Final Architecture

```text
                    GitHub
                       |
                       v
               AWS CodePipeline
                       |
                       v
                 AWS CodeBuild
                       |
                       v
              Docker Image Build
                       |
                       v
                  Amazon ECR
                       |
                       v
                Amazon EKS
                       |
              Kubernetes Deployment
                       |
                       v
                 Application Pods
                       |
                       v
              Kubernetes Service
                       |
                       v
              AWS Load Balancer
                       |
                       v
                    User

                       ^
                       |
                Amazon CloudWatch
             Logs / Monitoring / Metrics
```

## Project Outcome

The Brain Tasks React/Vite application was successfully containerized using Docker and NGINX, stored in Amazon ECR, and deployed to Amazon EKS.

AWS CodeBuild and AWS CodePipeline were used to automate the CI/CD workflow, while Amazon CloudWatch provided centralized monitoring and logging.

The deployment was verified using the AWS Console, AWS CLI and Kubernetes commands.

## Author

**M. Akshaya**

GitHub:

https://github.com/Akshaya33-Devops

Deployment Repository:

https://github.com/Akshaya33-Devops/BRAIN-TASKS-APP-DEPLOYMENT

## Project Information

```text
Project : BRAIN-TASKS-APP-DEPLOYMENT
Region  : ap-south-1 (Mumbai)
Year    : 2026
Status  : Completed
```
