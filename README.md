# MERN Microservices – AWS EKS Orchestration, CI/CD & Scaling

A production-style DevOps implementation of a MERN microservices application using Docker, Amazon ECR, Jenkins, Amazon EKS, Helm, Kubernetes Horizontal Pod Autoscaling (HPA), and Amazon CloudWatch.

The project demonstrates an end-to-end flow from source-code commit through automated container build, image publishing, Helm deployment to EKS, application scaling, centralized observability, and CloudWatch alerting.

## Project Status

| Area | Status | Validation |
|---|---|---|
| GitHub source control | ✅ Complete | Fork maintained on `main` |
| Docker containerization | ✅ Complete | Frontend, Hello Service and Profile Service images built successfully |
| Local Docker integration | ✅ Complete | MERN application validated through Nginx |
| Amazon ECR | ✅ Complete | Three application images published |
| Jenkins CI | ✅ Complete | Docker build and ECR push pipeline successful |
| GitHub webhook | ✅ Complete | Git push automatically triggered Jenkins |
| Amazon EKS | ✅ Complete | `mern-eks-cluster` running in `ap-south-1` |
| Helm deployment | ✅ Complete | Application deployed through `helm upgrade --install` |
| Jenkins → EKS CD | ✅ Complete | Jenkins automatically updates the EKS deployment with Helm |
| HPA | ✅ Complete | Frontend, Hello Service and Profile Service configured for 2–4 replicas |
| EKS worker scaling | ✅ Complete | Managed node group running 3 `t3.small` workers |
| CloudWatch control-plane logging | ✅ Complete | All five EKS control-plane log types enabled |
| CloudWatch Observability | ✅ Complete | EKS observability add-on active |
| Centralized logging | ✅ Complete | Container Insights application, host, dataplane and performance logs available |
| CloudWatch alarm | ✅ Complete | Worker CPU alarm created at 70% |

## Architecture

```text
                         Developer
                             |
                             | git push
                             v
                          GitHub
                             |
                         Webhook
                             |
                             v
                          Jenkins
                             |
                +------------+------------+
                |                         |
          Docker Build              AWS CLI / Helm
                |                         |
                v                         v
          Amazon ECR                 Amazon EKS
        +------+------+          +---------+---------+
        |      |      |          |                   |
        v      v      v          v                   v
     frontend hello profile   Frontend Pods      Backend Pods
        |      |      |          |                   |
        +------+------+
               |
               v
            MongoDB

       Amazon CloudWatch Observability
       |        |        |        |
       v        v        v        v
   Application  Host  Dataplane  Performance
       + EKS control-plane logs
```

### Runtime request flow

```text
Internet
   |
   v
AWS LoadBalancer
   |
   v
Frontend Service (Nginx)
   |---------------------------|
   |                           |
   v                           v
hello-service             profile-service
   :3001                       :3002
                               |
                               v
                          MongoDB :27017
```

## Technology Stack

### Application

- React
- Node.js
- Express.js
- MongoDB
- Mongoose
- Axios

### DevOps and Cloud

- Git / GitHub
- Docker
- Nginx
- Jenkins
- AWS CLI
- Amazon ECR
- Amazon EKS
- Kubernetes
- Helm
- Kubernetes HPA
- Amazon CloudWatch
- Container Insights
- Fluent Bit

## Repository Structure

```text
SampleMERNwithMicroservices/
├── backend/
│   ├── helloService/
│   │   ├── Dockerfile
│   │   ├── .dockerignore
│   │   ├── index.js
│   │   ├── package.json
│   │   └── package-lock.json
│   └── profileService/
│       ├── Dockerfile
│       ├── .dockerignore
│       ├── index.js
│       ├── package.json
│       └── package-lock.json
├── frontend/
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── .dockerignore
│   ├── public/
│   ├── src/
│   ├── package.json
│   └── package-lock.json
├── helm/
│   └── mern-app/
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── .helmignore
│       └── templates/
│           ├── frontend.yaml
│           ├── frontend-hpa.yaml
│           ├── hello-service.yaml
│           ├── hello-service-hpa.yaml
│           ├── mongodb.yaml
│           ├── profile-service.yaml
│           └── profile-service-hpa.yaml
├── k8s/
│   ├── namespace.yaml
│   ├── mongodb.yaml
│   ├── hello-service.yaml
│   ├── profile-service.yaml
│   └── frontend.yaml
├── Jenkinsfile
├── .gitignore
└── README.md
```

## 1. Git and Version Control

Repository:

`https://github.com/raviveera2305/SampleMERNwithMicroservices`

The repository is maintained on the `main` branch. Changes are committed and pushed through Git.

```bash
git status
git add .
git commit -m "descriptive change message"
git push origin main
```

Sensitive files such as `.env` files, access keys and tokens are excluded from the repository.

## 2. Containerization

Each application component has its own Docker image.

### Frontend

The frontend uses a multi-stage Docker build:

1. Node.js builds the React production application.
2. Nginx serves the production build.
3. Nginx reverse-proxies API requests to the backend services.

### Backend services

- Hello Service listens on port `3001`.
- Profile Service listens on port `3002`.
- Profile Service receives `MONGO_URL` at runtime.

### Nginx API routing

The frontend uses relative API paths:

```text
/api/hello/
/api/profile/fetchUser
```

Nginx routes them internally:

```text
/api/hello/       -> hello-service:3001/
/api/profile/*    -> profile-service:3002/*
```

This keeps browser-side API calls independent of hard-coded backend host addresses.

## 3. Local Docker Validation

A Docker network named `mern-network` was used for local integration.

| Container | Image | Port | Purpose |
|---|---|---:|---|
| `frontend` | `frontend:1.0` | `8080:80` | React + Nginx |
| `hello-service` | `hello-service:1.0` | `3001:3001` | Hello API |
| `profile-service` | `profile-service:1.0` | `3002:3002` | Profile API |
| `mongodb` | `mongo:7` | `27017:27017` | Database |

Validated endpoints included:

```text
GET /                    -> {"msg":"Hello World"}
GET /health              -> {"status":"OK"}
GET /fetchUser           -> []
```

The complete frontend was successfully opened through `http://localhost:8080`.

## 4. Amazon ECR

AWS Region:

```text
ap-south-1
```

Application ECR repositories:

```text
526362561261.dkr.ecr.ap-south-1.amazonaws.com/hello-service
526362561261.dkr.ecr.ap-south-1.amazonaws.com/profile-service
526362561261.dkr.ecr.ap-south-1.amazonaws.com/frontend
```

The repositories contain application images with release and CI-generated tags.

Jenkins authenticates to ECR using the IAM role attached to the Jenkins EC2 instance rather than storing AWS access keys in the pipeline.

## 5. Jenkins CI/CD and GitHub Webhook

Jenkins job:

```text
MERN-ECR-CI-CD
```

The pipeline is stored as `Jenkinsfile` in the repository.

### Pipeline flow

```text
GitHub Push
    |
    v
Checkout
    |
    v
ECR Login
    |
    v
Build Docker Images
    |
    v
Push Images to ECR
    |
    v
Update kubeconfig for EKS
    |
    v
Helm upgrade --install
    |
    v
Kubernetes rollout validation
    |
    v
Deployment / Pod / Service / HPA validation
```

### Jenkins stages

- Checkout
- ECR Login
- Build Docker Images
- Push Images to ECR
- Deploy to EKS with Helm
- Validate EKS Deployment

The Jenkins EC2 instance uses an IAM role for AWS operations, avoiding long-lived AWS access keys in Jenkins credentials.

### GitHub webhook automation

The Jenkins job is configured with the GitHub webhook trigger. A test push to `main` automatically triggered Jenkins, validating the GitHub → Jenkins integration.

### End-to-end CI/CD validation

The final Jenkins pipeline completed with:

```text
Finished: SUCCESS
```

This validates automated image build, ECR publishing, Helm deployment and Kubernetes rollout checks from Jenkins.

## 6. Amazon EKS

Cluster:

```text
Name:    mern-eks-cluster
Region:  ap-south-1
Version: Kubernetes v1.34.10
```

Managed node group:

```text
Node group:      mern-workers
Instance type:   t3.small
Minimum nodes:   1
Maximum nodes:   3
Desired nodes:   3
```

Final validation showed three worker nodes in `Ready` state.

```bash
kubectl get nodes
```

## 7. Kubernetes Deployment

Namespace:

```text
mern-app
```

Workloads:

- `frontend`
- `hello-service`
- `profile-service`
- `mongodb`

Services:

- `frontend` → `LoadBalancer`
- `hello-service` → `ClusterIP`
- `profile-service` → `ClusterIP`
- `mongodb` → `ClusterIP`

The frontend was validated through the AWS LoadBalancer and returned HTTP `200 OK`. The browser displayed the application UI, including the Welcome, Hello World and Profile sections.

## 8. Helm Deployment

Helm version:

```text
v3.22.0
```

Chart:

```text
helm/mern-app
```

The chart contains deployments, services and HPA resources for the application components.

### Chart validation

```bash
helm lint helm/mern-app
helm template mern-app helm/mern-app
```

Both validation and manifest rendering were completed successfully.

### Automated deployment

Jenkins deploys the chart with:

```bash
helm upgrade --install mern-app ./helm/mern-app \
  --namespace mern-app \
  --create-namespace \
  --wait \
  --timeout 10m
```

This keeps the Kubernetes application deployment under Helm management and makes the deployment repeatable from the CI/CD pipeline.

## 9. Horizontal Scaling and HPA

The application services use Kubernetes Horizontal Pod Autoscaling.

Helm values configure:

```yaml
replicaCount:
  helloService: 2
  profileService: 2
  frontend: 2
  mongodb: 1
```

HPA configuration for the stateless services:

```yaml
autoscaling:
  helloService:
    enabled: true
    minReplicas: 2
    maxReplicas: 4
    targetCPUUtilizationPercentage: 70

  profileService:
    enabled: true
    minReplicas: 2
    maxReplicas: 4
    targetCPUUtilizationPercentage: 70

  frontend:
    enabled: true
    minReplicas: 2
    maxReplicas: 4
    targetCPUUtilizationPercentage: 70
```

MongoDB remains at one replica because a standalone MongoDB deployment should not be duplicated without a proper replicated database architecture.

### Live HPA validation

```bash
kubectl get hpa -n mern-app
```

The live cluster showed:

```text
NAME              MINPODS   MAXPODS   REPLICAS
frontend          2         4         2
hello-service     2         4         3
profile-service   2         4         3
```

The HPA also demonstrated an actual scale-up event during validation. `hello-service` reached approximately `157%` CPU against a `70%` target and the HPA increased the workload to four replicas.

After additional worker capacity became available, all requested replicas were scheduled successfully.

Final deployment validation:

```text
frontend          2/2
hello-service     3/3
mongodb           1/1
profile-service   3/3
```

## 10. EKS Worker Scaling

The initial worker node reached Kubernetes pod capacity while HPA was scaling application workloads. Kubernetes reported:

```text
Too many pods
```

The managed node group was increased to three `t3.small` workers:

```bash
eksctl scale nodegroup \
  --cluster mern-eks-cluster \
  --region ap-south-1 \
  --name mern-workers \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 3
```

The final cluster contained three `Ready` worker nodes, allowing the HPA-managed application replicas to be scheduled successfully.

## 11. CloudWatch Monitoring and Logging

### EKS control-plane logging

All five EKS control-plane log types were enabled:

```text
api
audit
authenticator
controllerManager
scheduler
```

Verification:

```bash
aws eks describe-cluster \
  --name mern-eks-cluster \
  --region ap-south-1 \
  --query 'cluster.logging.clusterLogging'
```

The configuration returned all five types as enabled.

### CloudWatch Observability

The Amazon EKS add-on:

```text
amazon-cloudwatch-observability
```

was installed and verified as `ACTIVE`.

The `amazon-cloudwatch` namespace contained the observability controller, CloudWatch agents and Fluent Bit collectors running across the worker nodes.

### Centralized logs

CloudWatch Container Insights log groups were verified for:

```text
/aws/containerinsights/mern-eks-cluster/application
/aws/containerinsights/mern-eks-cluster/dataplane
/aws/containerinsights/mern-eks-cluster/host
/aws/containerinsights/mern-eks-cluster/performance
```

Application logs were also validated from Kubernetes, for example:

```bash
kubectl logs deployment/hello-service -n mern-app --tail=20
```

with the application reporting:

```text
Server is running on port 3001
```

### CloudWatch alarm

A CloudWatch alarm was created for the EKS worker Auto Scaling Group:

```text
Alarm:     MERN-EKS-Worker-CPU-High
Metric:    CPUUtilization
Statistic: Average
Period:    5 minutes
Condition: Greater than 70%
Datapoints: 1 out of 1
```

The alarm provides threshold-based monitoring for worker CPU utilization. Notifications were intentionally not attached because the core assignment requires alarm configuration; SNS/ChatOps can be added as an extension.

## 12. Security and Configuration Practices

- AWS access keys are not stored in `Jenkinsfile`.
- Jenkins uses an EC2 IAM role for AWS authentication.
- The CloudWatch agent uses a dedicated IAM service-account role.
- `.env` and environment-specific secret files are excluded from Docker build contexts.
- Secrets and tokens must never be committed to GitHub.
- Runtime configuration such as `MONGO_URL` is supplied at deployment time.
- Production deployments should use HTTPS and a stable DNS name.
- The academic Jenkins environment should be restricted and protected with HTTPS in a production deployment.

## 13. Troubleshooting and Lessons Learned

### Jenkins resource pressure

The Jenkins host is a small `t2.micro`. Frontend dependency installation required additional memory, so a 2 GiB swap file was configured on the Jenkins host. The pipeline subsequently completed successfully.

### Kubernetes pod capacity

HPA scaling exposed the pod-capacity limit of the initial worker configuration. Increasing the EKS managed node group to three `t3.small` workers resolved the scheduling constraint.

### Helm ownership

The first Helm release was created in the wrong namespace. It was removed and the release was recreated through Jenkins using the correct `mern-app` namespace. This established consistent Helm ownership metadata and allowed automated upgrades to succeed.

## 14. Validation Commands

Useful commands for reproducing the final validation:

```bash
# EKS
kubectl get nodes

# Application workloads
kubectl get deployments -n mern-app
kubectl get pods -n mern-app -o wide
kubectl get svc -n mern-app

# HPA
kubectl get hpa -n mern-app

# Helm
helm list --all-namespaces
helm status mern-app -n mern-app

# CloudWatch observability
kubectl get pods -n amazon-cloudwatch

# Application logs
kubectl logs deployment/hello-service -n mern-app --tail=20
```

## 15. Final Evidence Checklist

Recommended evidence for the academic submission:

1. GitHub repository and commit history.
2. Dockerfiles and local Docker application validation.
3. Amazon ECR repositories with application images.
4. Jenkins successful CI/CD pipeline.
5. GitHub webhook configuration and automatic Jenkins build.
6. EKS cluster with three `Ready` worker nodes.
7. Kubernetes deployments, pods and services in `mern-app`.
8. Helm release and successful Jenkins Helm deployment.
9. Browser showing the application through the AWS LoadBalancer.
10. HPA output showing 2–4 replica ranges and live replica counts.
11. Evidence of HPA scale-up under high CPU.
12. CloudWatch Observability pods running across worker nodes.
13. CloudWatch Container Insights log groups.
14. CloudWatch `MERN-EKS-Worker-CPU-High` alarm.
15. Final Jenkins build showing `Finished: SUCCESS`.

## 16. Final Validation Summary

The completed implementation provides the following automated DevOps flow:

```text
Developer Commit
      |
      v
GitHub main
      |
      | webhook
      v
Jenkins
      |
      +--> Docker Build
      |
      +--> Amazon ECR Push
      |
      +--> EKS kubeconfig
      |
      +--> Helm Deployment
      |
      +--> Kubernetes Rollout Validation
      |
      +--> HPA / Service / Pod Validation
      |
      v
Amazon EKS
      |
      +--> 3 worker nodes
      +--> 2–4 replicas for stateless services
      +--> LoadBalancer frontend
      +--> MongoDB backend
      |
      v
Amazon CloudWatch
      |
      +--> Control-plane logs
      +--> Container Insights
      +--> Centralized logs
      +--> Worker CPU alarm
```

The project now demonstrates containerization, CI/CD, EKS orchestration, Helm-based deployment, horizontal autoscaling, worker-node scaling, centralized monitoring/logging, and CloudWatch alerting in one integrated workflow.
