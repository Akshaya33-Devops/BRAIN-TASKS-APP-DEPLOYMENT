# Brain Tasks App – Deployment on AWS (Docker · ECR · EKS · GitHub Actions · CloudWatch)

End-to-end deployment of a React/Vite single-page application ("Brain Tasks") using containerization, Kubernetes and AWS DevOps services.

The app is packaged into a Docker image, pushed to Amazon ECR by a GitHub Actions workflow, deployed to an Amazon EKS cluster, exposed to the internet through a Kubernetes LoadBalancer service, and monitored with Amazon CloudWatch.

- **AWS Region:** `ap-south-1` (Asia Pacific – Mumbai)
- **Prepared by:** M. Akshaya
- **Application source:** https://github.com/Vennilavanguvi/Brain-Tasks-App

---

## Architecture

```
GitHub Repository
       │
       ▼
GitHub Actions (build Docker image)
       │  push image
       ▼
Amazon ECR (brain-tasks-app)
       │  pull image
       ▼
Amazon EKS Cluster (brain-tasks-cluster)
       │                         ┆ logs / metrics
       ▼                         ▼
Kubernetes Deployment      Amazon CloudWatch
(brain-tasks-app, 2 replicas)   (Logs, Container Insights)
       │  ◀── scales ──▶ Horizontal Pod Autoscaler (2–5, CPU 70%)
       ▼
Kubernetes Service (LoadBalancer)
       │
       ▼
AWS Elastic Load Balancer  ──HTTP──▶  End User / Browser
```

## Tech Stack

| Area | Tool |
|---|---|
| Application | React + Vite (pre-built `dist/`) |
| Web server | NGINX (alpine) |
| Containerization | Docker |
| Container registry | Amazon ECR |
| Orchestration | Amazon EKS (Kubernetes 1.34) |
| CI/CD | GitHub Actions |
| Monitoring / logging | Amazon CloudWatch (Container Insights, Fluent Bit) |
| Scaling | Kubernetes Horizontal Pod Autoscaler |

## Repository Structure

```
BRAIN-TASKS-APP-DEPLOYMENT/
├── .github/workflows/   # GitHub Actions CI/CD workflow
├── dist/                # Pre-built React/Vite production build
│   ├── assets/
│   ├── index.html
│   └── vite.svg
├── Dockerfile
├── deployment.yaml      # Kubernetes Deployment
├── service.yaml         # Kubernetes LoadBalancer Service
└── README.md
```

> The application is supplied as a pre-built production distribution, so there is no `package.json` and no build step for the app itself.

## Dockerfile

```dockerfile
FROM nginx:alpine
COPY dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

NGINX listens on port **80** inside the container.

## Run Locally with Docker

```bash
# Build the image
docker build -t brain-tasks-app:latest .

# Run the container (host 3000 -> container 80)
docker run -d --name brain-tasks-app-container -p 3000:80 brain-tasks-app:latest
```

Open http://localhost:3000

## Push the Image to Amazon ECR

The workflow does this automatically, but the manual flow is:

```bash
aws ecr get-login-password --region ap-south-1 \
  | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com

docker tag brain-tasks-app:latest <AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/brain-tasks-app:latest
docker push <AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/brain-tasks-app:latest
```

Replace `<AWS_ACCOUNT_ID>` with your own account ID.

## Deploy to Amazon EKS

```bash
# Connect kubectl to the cluster
aws eks update-kubeconfig --region ap-south-1 --name brain-tasks-cluster

# Apply the manifests
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Autoscaling: 2–5 replicas at 70% CPU
kubectl autoscale deployment brain-tasks-app --cpu-percent=70 --min=2 --max=5
```

### Verify

```bash
kubectl get deployment,service,pods -o wide
kubectl get hpa
kubectl top pods
```

Get the external endpoint from `kubectl get service` (the `EXTERNAL-IP` column of `brain-tasks-service`), then test it:

```powershell
Invoke-WebRequest http://<LOAD-BALANCER-DNS>
```

A `200 OK` response confirms the app is reachable.

## CI/CD – GitHub Actions

On each run the workflow:

1. Checks out the source code
2. Builds the Docker image
3. Authenticates with Amazon ECR
4. Pushes the image to ECR
5. Deploys the application to Amazon EKS

**Why GitHub Actions instead of AWS CodeBuild?** CodeBuild was the original plan, but the AWS account hit a concurrent-build quota limit. A quota increase request (one concurrent Linux/Small build) was not approved because the account had insufficient usage history. GitHub Actions replaced it, with GUVI's approval.

## Monitoring – Amazon CloudWatch

Container Insights ships Kubernetes logs and metrics to CloudWatch Logs via Fluent Bit. The log groups are:

- `/aws/containerinsights/brain-tasks-cluster/application`
- `/aws/containerinsights/brain-tasks-cluster/dataplane`
- `/aws/containerinsights/brain-tasks-cluster/host`

```bash
aws logs describe-log-groups --region ap-south-1 \
  --query "logGroups[*].logGroupName" --output table
```

## Troubleshooting Highlights

| Issue | Fix |
|---|---|
| Image not available in ECR | Built and pushed `brain-tasks-app` to ECR, verified digest with the AWS CLI |
| `kubectl` not connected to EKS | `aws eks update-kubeconfig --region ap-south-1 --name brain-tasks-cluster` |
| CloudWatch `AccessDeniedException` from Fluent Bit | Created an IAM policy and role, installed the EKS Pod Identity Agent, associated the `cloudwatch-agent` service account with the role, then ran `kubectl rollout restart daemonset/fluent-bit -n amazon-cloudwatch` |
| Log groups / streams not visible right away | Verified the CloudWatch observability add-on and Fluent Bit config, generated traffic through the load balancer, and waited for ingestion |
| `kubectl get hpa` returned no resources | Created the HPA with `kubectl autoscale` (2–5 replicas, 70% CPU) |
| CodeBuild concurrent build quota not approved | Switched CI/CD to GitHub Actions |

## Project Status

- [x] Application source cloned and `dist/` verified
- [x] Dockerfile created, image built, container tested on `localhost:3000`
- [x] Image pushed to Amazon ECR
- [x] EKS cluster provisioned and active
- [x] Kubernetes Deployment and LoadBalancer Service created, app reachable externally
- [x] GitHub Actions workflow: GitHub → GitHub Actions → ECR → EKS
- [x] CloudWatch Container Insights log groups active
- [x] Horizontal Pod Autoscaler configured (2–5 replicas, 70% CPU)

## Cleanup

To avoid ongoing AWS charges when you are done:

```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
kubectl delete hpa brain-tasks-app
# Then delete the EKS cluster/node group and the ECR repository from the AWS Console or CLI
```

## Acknowledgements

Application source by [Vennilavanguvi](https://github.com/Vennilavanguvi/Brain-Tasks-App). Deployment work completed as part of the GUVI DevOps Engineering program.
