# MERN Microservices – AWS EKS Orchestration & Scaling

A DevOps implementation of a MERN microservices application containerized with Docker and deployed to Amazon EKS. The project demonstrates source control, Docker image creation, Amazon ECR, Jenkins CI, GitHub webhook automation, Kubernetes, Helm, horizontal scaling, and CloudWatch monitoring/logging.

## Project Status

| Component | Status | Validation |
|---|---|---|
| GitHub repository | ✅ Complete | Fork maintained on `main` |
| Docker containerization | ✅ Complete | Frontend, Hello Service and Profile Service images built and validated |
| Local Docker integration | ✅ Complete | Application validated through Nginx at `http://localhost:8080` |
| Amazon ECR | ✅ Complete | Three application images pushed to ECR |
| Jenkins CI | ✅ Complete | Pipeline builds and pushes images to ECR |
| GitHub webhook | ✅ Complete | Git push automatically triggered Jenkins Build #3 successfully |
| Amazon EKS | ✅ Complete | `mern-eks-cluster` running in `ap-south-1` |
| Kubernetes deployment | ✅ Complete | Four workloads and services running in `mern-app` |
| Helm | ✅ Complete | `mern-app` release deployed and upgraded to revision 2 |
| Horizontal scaling | ✅ Complete | Frontend, Hello Service and Profile Service scaled to 2 replicas |
| EKS worker scaling | ✅ Complete | Managed node group scaled from 1 to 2 `t3.small` nodes |
| CloudWatch control-plane logging | ✅ Complete | API, audit, authenticator, controller manager and scheduler enabled |
| CloudWatch Observability | ✅ Complete | `amazon-cloudwatch-observability` add-on is `ACTIVE` |
| Centralized logging | ✅ Complete | CloudWatch application, host, dataplane and performance log groups present |
| Automated EKS deployment from Jenkins | ⚠️ Not implemented | EKS/Helm deployment was performed and validated separately |
| HPA | ⚠️ Not implemented | Fixed replica scaling demonstrated with Helm |

## 1. Architecture

```text
                         GitHub
                           |
                           | Push
                           v
                        Jenkins
                           |
                    Docker Build / Push
                           |
                           v
                    Amazon ECR
             +-------------+-------------+
             |             |             |
             v             v             v
        frontend     hello-service   profile-service
             |             |             |
             +-------------+-------------+
                           |
                           v
                     Amazon EKS
                           |
                    Helm-managed apps
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
        Frontend Pods  Hello Pods   Profile Pods
                                         |
                                         v
                                     MongoDB

                    CloudWatch Observability
                    /        |        |       \
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
   |--------------------------|
   |                          |
   v                          v
hello-service            profile-service
   :3001                      :3002
                              |
                              v
                         MongoDB :27017
```

## 2. Technology Stack

### Application

- React
- Node.js
- Express.js
- MongoDB
- Mongoose
- Axios

### DevOps / Cloud

- Git / GitHub
- Docker
- Nginx
- Jenkins
- AWS CLI
- Amazon ECR
- Amazon EKS
- Kubernetes
- Helm
- Amazon CloudWatch

## 3. Repository Structure

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
│           ├── hello-service.yaml
│           ├── mongodb.yaml
│           └── profile-service.yaml
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

## 4. Git and Version Control

Repository:

`https://github.com/raviveera2305/SampleMERNwithMicroservices`

The project is maintained on the `main` branch. Changes are committed and pushed through Git.

Typical workflow:

```bash
git status
git add .
git commit -m "descriptive change message"
git push origin main
```

No passwords, access keys, GitHub tokens, or `.env` files are stored in the repository.

## 5. Containerization

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

### Frontend API routing

The React application uses relative API paths:

```text
/api/hello/
/api/profile/fetchUser
```

Nginx routes them internally:

```text
/api/hello/       -> hello-service:3001/
/api/profile/*    -> profile-service:3002/*
```

This avoids hard-coded browser-side `localhost` backend URLs.

## 6. Local Docker Validation

A Docker network named `mern-network` was used for local integration.

| Container | Image | Port | Purpose |
|---|---|---:|---|
| `frontend` | `frontend:1.0` | `8080:80` | React + Nginx |
| `hello-service` | `hello-service:1.0` | `3001:3001` | Hello API |
| `profile-service` | `profile-service:1.0` | `3002:3002` | Profile API |
| `mongodb` | `mongo:7` | `27017:27017` | Database |

Validated responses:

```text
GET http://localhost:3001/
{"msg":"Hello World"}

GET http://localhost:3001/health
{"status":"OK"}

GET http://localhost:3002/health
{"status":"OK"}

GET http://localhost:3002/fetchUser
[]
```

The complete application was also opened successfully at `http://localhost:8080`.

## 7. Amazon ECR

AWS Region:

```text
ap-south-1
```

Three ECR repositories were created and populated with application images:

```text
526362561261.dkr.ecr.ap-south-1.amazonaws.com/hello-service
526362561261.dkr.ecr.ap-south-1.amazonaws.com/profile-service
526362561261.dkr.ecr.ap-south-1.amazonaws.com/frontend
```

Images validated in ECR included the `1.0` release tags and CI-generated `latest` tags.

The Jenkins EC2 instance uses its IAM role for ECR authentication; AWS access keys are not stored in the Jenkinsfile.

## 8. Jenkins CI and GitHub Webhook

Jenkins job:

```text
MERN-ECR-CI-CD
```

The Jenkins pipeline is stored in the repository as `Jenkinsfile`.

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
Build frontend + backend Docker images
    |
    v
Push images to Amazon ECR
```

### Jenkins stages

- Checkout
- ECR Login
- Build Docker Images
- Push Images to ECR

The Jenkins EC2 instance uses the IAM role `Jenkins-ECR-Role` instead of storing AWS access keys in Jenkins credentials.

### Automatic GitHub trigger

The Jenkins job is configured with the GitHub webhook trigger. A test empty commit was pushed to `main`, which automatically triggered **Jenkins Build #3**, and the build completed successfully.

This validates the GitHub → Jenkins webhook integration.

> **Scope note:** The current Jenkinsfile automates image build and ECR publishing. Kubernetes/Helm deployment was completed separately on EKS and is documented below.

## 9. Amazon EKS

Cluster:

```text
Name:    mern-eks-cluster
Region:  ap-south-1
Version: Kubernetes v1.34.10
```

The cluster was created with `eksctl` using a managed node group:

```text
Node group:      mern-workers
Instance type:   t3.small
Minimum nodes:   1
Maximum nodes:   2
```

The cluster was validated with:

```bash
kubectl get nodes
```

Final validation showed two worker nodes in `Ready` state, both running Kubernetes `v1.34.10`.

## 10. Kubernetes Deployment

Namespace:

```text
mern-app
```

Workloads deployed:

- `frontend`
- `hello-service`
- `profile-service`
- `mongodb`

Services:

- `frontend` → `LoadBalancer`
- `hello-service` → `ClusterIP`
- `profile-service` → `ClusterIP`
- `mongodb` → `ClusterIP`

The frontend LoadBalancer successfully returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.27.5
```

The application was also successfully opened through the AWS LoadBalancer and displayed the Welcome, Hello World and Profile sections.

## 11. Helm Deployment

Helm version used:

```text
v3.22.0
```

Chart:

```text
helm/mern-app
```

The chart contains:

```text
Chart.yaml
values.yaml
.helmignore
templates/
├── frontend.yaml
├── hello-service.yaml
├── mongodb.yaml
└── profile-service.yaml
```

### Validation

```bash
helm lint helm/mern-app
```

The chart passed lint validation.

The rendered Kubernetes manifests were checked with:

```bash
helm template mern-app helm/mern-app
```

### Install

```bash
helm install mern-app helm/mern-app
```

Helm reported:

```text
STATUS: deployed
REVISION: 1
```

### Upgrade

The replica values were later changed and applied with:

```bash
helm upgrade mern-app helm/mern-app
```

The release advanced to:

```text
REVISION: 2
```

## 12. Horizontal Scaling

The stateless application services were scaled from 1 to 2 replicas through Helm values:

```yaml
replicaCount:
  helloService: 2
  profileService: 2
  frontend: 2
  mongodb: 1
```

MongoDB was intentionally kept at one replica because simply duplicating a standalone MongoDB deployment does not provide database replication or safe shared state.

The first scaling attempt exposed the pod-capacity limit of the single worker node. Kubernetes reported:

```text
0/1 nodes are available: 1 Too many pods
```

The managed node group was therefore scaled from 1 to 2 worker nodes:

```bash
eksctl scale nodegroup \
  --cluster mern-eks-cluster \
  --name mern-workers \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 2 \
  --region ap-south-1
```

After the second worker became Ready, all application replicas were scheduled successfully.

Final deployment validation:

```text
frontend          2/2
hello-service     2/2
mongodb           1/1
profile-service   2/2
```

The final pod placement showed application replicas distributed across both EKS worker nodes.

> **Scaling note:** This project demonstrates fixed horizontal replica scaling through Helm. An HPA was not implemented.

## 13. CloudWatch Monitoring and Logging

### 13.1 EKS Control-Plane Logging

All five EKS control-plane log types were enabled:

```text
api
 audit
authenticator
controllerManager
scheduler
```

The configuration was verified using:

```bash
aws eks describe-cluster \
  --name mern-eks-cluster \
  --region ap-south-1 \
  --query 'cluster.logging.clusterLogging'
```

The result showed all five types with `enabled: true`.

### 13.2 CloudWatch Observability Add-on

The Amazon EKS add-on:

```text
amazon-cloudwatch-observability
```

was installed and verified as:

```text
ACTIVE
```

The add-on uses an IAM service-account role created for the CloudWatch agent.

### 13.3 CloudWatch agents and Fluent Bit

The `amazon-cloudwatch` namespace was validated with:

```bash
kubectl get pods -n amazon-cloudwatch
```

The final state included:

```text
amazon-cloudwatch-observability-controller-manager   Running
cloudwatch-agent                                     Running (2 pods)
fluent-bit                                           Running (2 pods)
```

The agents and Fluent Bit collectors were therefore running across both worker nodes.

### 13.4 Application logging

Application logs were verified from Kubernetes:

```bash
kubectl logs deployment/hello-service -n mern-app --tail=20
```

Example validated log:

```text
Server is running on port 3001
```

### 13.5 CloudWatch log groups

CloudWatch contained the following EKS Container Insights log groups:

```text
/aws/containerinsights/mern-eks-cluster/application
/aws/containerinsights/mern-eks-cluster/dataplane
/aws/containerinsights/mern-eks-cluster/host
/aws/containerinsights/mern-eks-cluster/performance
```

These provide centralized application/container, host, dataplane and performance telemetry.

## 14. Security and Configuration Practices

- AWS access keys are not stored in the Jenkinsfile.
- Jenkins uses an EC2 IAM role for AWS operations.
- The CloudWatch agent uses a dedicated IAM service-account role.
- `.env` and environment-specific secret files are excluded through `.dockerignore`.
- Secrets and tokens must never be committed to GitHub.
- Runtime configuration such as `MONGO_URL` is supplied through Kubernetes environment variables.
- Production deployments should use HTTPS and a stable DNS name rather than an HTTP AWS LoadBalancer hostname.
- The Jenkins web interface was exposed on port `8080` for the academic environment; production deployments should restrict access and use HTTPS.

## 15. Troubleshooting Notes

### Jenkins temporary resource pressure

The Jenkins EC2 instance is a small `t2.micro`. Frontend dependency installation initially required additional memory. A 2 GiB swap file was configured on the Jenkins host, after which the pipeline completed successfully.

### EKS pod scheduling

When the application was scaled to two replicas, the single worker reached its maximum pod capacity. Kubernetes reported `Too many pods`. The managed node group was increased to two nodes, after which all replicas became Ready.

### Helm resource ownership

The initial Kubernetes resources were created with `kubectl apply`. Before Helm installation, those manually managed application resources were removed and recreated through Helm so that Helm could manage the release cleanly.

## 16. Evidence Checklist

Recommended screenshots/evidence for final submission:

1. GitHub repository and commit history.
2. Docker images and local application validation.
3. Amazon ECR repositories containing pushed images.
4. Jenkins successful pipeline.
5. GitHub webhook configuration and successful automatic Jenkins build.
6. EKS cluster and two Ready worker nodes.
7. Kubernetes workloads and services.
8. Helm release with `STATUS: deployed` and revision 2 after upgrade.
9. Browser showing the application through the AWS LoadBalancer.
10. Scaling evidence showing 2/2 replicas for frontend, Hello Service and Profile Service.
11. CloudWatch Observability pods running on both worker nodes.
12. CloudWatch Container Insights log groups.
13. Final end-to-end application validation.

## 17. Final Validation Summary

The completed implementation demonstrates the following end-to-end flow:

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
      v
Docker Build
      |
      v
Amazon ECR
      |
      v
Amazon EKS
      |
      v
Helm Release
      |
      +--> Frontend (2 replicas)
      +--> Hello Service (2 replicas)
      +--> Profile Service (2 replicas)
      +--> MongoDB (1 replica)
      |
      v
AWS LoadBalancer
      |
      v
Browser

CloudWatch
  +--> EKS control-plane logs
  +--> Application logs
  +--> Host logs
  +--> Dataplane logs
  +--> Performance telemetry
```

The application was successfully deployed, accessed through the AWS LoadBalancer, scaled across two EKS worker nodes, and integrated with CloudWatch monitoring and centralized logging.

## 18. Future Improvements

For a production-grade extension, the project could add:

- Jenkins-driven Helm deployment to EKS after successful image publishing.
- Immutable image tags based on Git commit SHA instead of relying on `latest`.
- Horizontal Pod Autoscaler using CPU/memory metrics.
- MongoDB Atlas or a managed database instead of a single in-cluster MongoDB pod.
- Kubernetes Secrets or AWS Secrets Manager for sensitive configuration.
- HTTPS with a stable DNS name and AWS Certificate Manager.
- CloudWatch alarms and dashboards for operational thresholds.
- SNS/Slack/Teams/Telegram notifications for CI/CD and operational events.

---

**Repository:** `https://github.com/raviveera2305/SampleMERNwithMicroservices`
