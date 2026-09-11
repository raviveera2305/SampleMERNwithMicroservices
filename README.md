# MERN Microservices – AWS EKS Orchestration & Scaling

A production-oriented DevOps implementation of a MERN application composed of independent frontend and backend microservices. The project is being progressively containerized and prepared for deployment on AWS using Amazon ECR, Jenkins CI/CD, Amazon EKS, Helm, Kubernetes scaling, CloudWatch monitoring, and centralized logging.

> **Project status:** Containerization and local Docker validation are complete. AWS ECR, Jenkins, EKS, Helm, scaling, monitoring, and centralized logging will be documented here as they are implemented and validated.

## 1. Project Overview

This project demonstrates how a Node.js/Express and React application can be transformed from a locally runnable MERN application into a containerized, cloud-native workload suitable for orchestration on Amazon EKS.

The application contains:

- **Frontend:** React application served through Nginx.
- **Hello Service:** Node.js/Express microservice exposing greeting and health endpoints.
- **Profile Service:** Node.js/Express microservice connected to MongoDB for profile data.
- **MongoDB:** Database service used by the Profile Service.

The implementation is designed around independent services, container isolation, service-to-service networking, repeatable deployments, and future horizontal scaling.

## 2. Project Objectives

The implementation is aligned with the following DevOps and cloud-native objectives:

1. Maintain the application in Git and GitHub.
2. Containerize each application component using Docker.
3. Build and publish versioned images to Amazon ECR.
4. Implement Jenkins-based CI/CD for automated image builds and deployments.
5. Deploy the application to Amazon EKS using Kubernetes and Helm.
6. Demonstrate application scaling using Kubernetes mechanisms.
7. Configure AWS monitoring and centralized logging.
8. Validate application availability and document implementation evidence.

## 3. Current Implementation Status

| Component | Status | Notes |
|---|---|---|
| GitHub repository | ✅ Completed | Forked repository maintained under the project account |
| Git version control | ✅ Completed | Changes committed and pushed to `main` |
| Frontend Docker image | ✅ Completed | `frontend:1.0` validated locally |
| Hello Service Docker image | ✅ Completed | `hello-service:1.0` validated locally |
| Profile Service Docker image | ✅ Completed | `profile-service:1.0` validated locally |
| MongoDB container | ✅ Completed | Validated on the local Docker network |
| Nginx reverse proxy | ✅ Completed | Frontend routes API requests to backend services |
| Local container integration | ✅ Completed | Full application validated at `http://localhost:8080` |
| Amazon ECR | ⏳ Pending | To be implemented |
| Jenkins CI/CD | ⏳ Pending | To be implemented |
| Amazon EKS | ⏳ Pending | To be implemented |
| Helm deployment | ⏳ Pending | To be implemented |
| Kubernetes scaling / HPA | ⏳ Pending | To be implemented |
| CloudWatch monitoring | ⏳ Pending | To be implemented |
| Centralized logging | ⏳ Pending | To be implemented |
| Final cloud validation | ⏳ Pending | To be completed after AWS deployment |

## 4. Architecture

### 4.1 Current Local Architecture

```text
                         Browser
                            |
                            | http://localhost:8080
                            v
                    +------------------+
                    | Frontend Container|
                    | React + Nginx    |
                    | Port 80          |
                    +--------+---------+
                             |
                 +-----------+-----------+
                 |                       |
        /api/hello/              /api/profile/
                 |                       |
                 v                       v
        +----------------+      +------------------+
        | hello-service  |      | profile-service  |
        | Node/Express   |      | Node/Express     |
        | Port 3001      |      | Port 3002        |
        +----------------+      +--------+---------+
                                         |
                                         | MongoDB
                                         v
                                  +--------------+
                                  |   MongoDB    |
                                  |   Port 27017 |
                                  +--------------+

                    Docker Network: mern-network
```

### 4.2 Target Cloud Architecture

The final architecture will extend the same service boundaries into Kubernetes on Amazon EKS:

```text
                         Internet / User
                                |
                                v
                     Kubernetes Load Balancer
                                |
                                v
                       Frontend Service
                                |
                +---------------+---------------+
                |                               |
                v                               v
        Hello Service Pods              Profile Service Pods
                |                               |
                |                               v
                |                        MongoDB / DB Layer
                |
                +-------------------------------+

                 Amazon EKS Cluster
                         |
                  Helm-managed workloads
                         |
              +----------+----------+
              |                     |
        CloudWatch Metrics    Centralized Logs
```

## 5. Technology Stack

### Application

- React
- Node.js
- Express.js
- MongoDB
- Mongoose
- Axios

### Containerization

- Docker
- Docker Desktop
- Nginx
- Docker bridge network

### Planned AWS / DevOps Platform

- AWS CLI
- Amazon ECR
- Jenkins
- Amazon EKS
- Kubernetes
- Helm
- CloudWatch

## 6. Repository Structure

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
├── .gitignore
└── README.md
```

## 7. Git and Version Control

The project is maintained in Git and hosted on GitHub.

Repository:

**https://github.com/raviveera2305/SampleMERNwithMicroservices**

Development workflow:

```bash
git status
git add .
git commit -m "descriptive change message"
git push origin main
```

Sensitive credentials, environment secrets, and local configuration files must not be committed to the repository.

## 8. Containerization

Each application component has its own Docker image.

### 8.1 Frontend Dockerfile

The frontend uses a multi-stage Docker build:

1. Node.js builds the React production bundle.
2. Nginx serves the generated static files.
3. Nginx reverse-proxies API requests to the backend containers.

This keeps the runtime image lightweight and separates the build environment from production serving.

### 8.2 Hello Service Dockerfile

The Hello Service runs on Node.js Alpine and exposes port `3001`.

### 8.3 Profile Service Dockerfile

The Profile Service runs on Node.js Alpine and exposes port `3002`. MongoDB connection information is supplied at runtime rather than baked into the image.

### 8.4 Docker Ignore Rules

The services use `.dockerignore` files to keep unnecessary or sensitive local files out of images, including:

```text
node_modules
npm-debug.log
.git
.gitignore
.env
.env.*
README.md
```

## 9. Frontend API Routing

The frontend no longer depends on browser-side `localhost` backend URLs. API requests use relative paths:

```text
/api/hello/
/api/profile/fetchUser
```

Nginx routes these requests internally:

```text
/api/hello/              -> hello-service:3001/
/api/profile/*           -> profile-service:3002/*
```

This design allows the same frontend image to work in a containerized environment without hard-coding the backend host into the browser application.

## 10. Local Docker Network

The application components were validated on a shared Docker network:

```text
mern-network
```

The local container topology is:

| Container | Image | Port | Purpose |
|---|---|---:|---|
| `frontend` | `frontend:1.0` | `8080:80` | React application + Nginx |
| `hello-service` | `hello-service:1.0` | `3001:3001` | Hello API |
| `profile-service` | `profile-service:1.0` | `3002:3002` | Profile API |
| `mongodb` | `mongo:7` | `27017:27017` | MongoDB database |

## 11. Local Deployment

### 11.1 Create the Docker Network

```powershell
docker network create mern-network
```

### 11.2 Start MongoDB

```powershell
docker run -d `
  --name mongodb `
  --network mern-network `
  -p 27017:27017 `
  mongo:7
```

### 11.3 Start Hello Service

```powershell
docker run -d `
  --name hello-service `
  --network mern-network `
  -e PORT=3001 `
  -p 3001:3001 `
  hello-service:1.0
```

### 11.4 Start Profile Service

```powershell
docker run -d `
  --name profile-service `
  --network mern-network `
  -e PORT=3002 `
  -e MONGO_URL=mongodb://mongodb:27017/streamingapp `
  -p 3002:3002 `
  profile-service:1.0
```

### 11.5 Start Frontend

```powershell
docker run -d `
  --name frontend `
  --network mern-network `
  -p 8080:80 `
  frontend:1.0
```

## 12. Local Validation

The following checks were successfully completed during local container validation.

### Hello Service

```text
GET http://localhost:3001/
Response: {"msg":"Hello World"}
```

```text
GET http://localhost:3001/health
Response: {"status":"OK"}
```

### Profile Service

```text
GET http://localhost:3002/health
Response: {"status":"OK"}
```

```text
GET http://localhost:3002/fetchUser
Response: []
```

### Frontend

The complete containerized application was successfully opened in a browser at:

```text
http://localhost:8080
```

The validated UI displayed the application welcome message, the Hello Service response, and the Profile section.

## 13. Amazon ECR

**Status: ⏳ Pending**

The following image repositories are planned:

```text
hello-service
profile-service
frontend
```

After implementation, this section will contain the actual ECR repository URIs, authentication commands, image tagging commands, push commands, and validation evidence.

## 14. Jenkins CI/CD

**Status: ⏳ Pending**

The planned CI/CD pipeline will:

```text
Git Push
   |
   v
Jenkins Trigger
   |
   +--> Checkout Source
   |
   +--> Build Docker Images
   |
   +--> Authenticate to ECR
   |
   +--> Push Versioned Images
   |
   +--> Deploy / Update Kubernetes Workloads
   |
   v
EKS
```

Actual Jenkins configuration, credentials strategy, pipeline stages, trigger configuration, and evidence will be documented after implementation.

## 15. Amazon EKS and Kubernetes

**Status: ⏳ Pending**

The final deployment is planned for Amazon EKS with separate Kubernetes workloads and services for:

- Frontend
- Hello Service
- Profile Service
- Database layer

Cluster configuration, node groups, namespaces, services, deployments, probes, resources, and ingress/load-balancing configuration will be documented after implementation.

## 16. Helm Deployment

**Status: ⏳ Pending**

Helm will be used to package and manage the Kubernetes deployment.

The planned chart structure is:

```text
helm/
└── streamingapp/
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
        ├── frontend-deployment.yaml
        ├── frontend-service.yaml
        ├── hello-deployment.yaml
        ├── hello-service.yaml
        ├── profile-deployment.yaml
        ├── profile-service.yaml
        └── ...
```

The final README will document `helm install`, upgrades, rollback procedures, values, and validation commands.

## 17. Scaling

**Status: ⏳ Pending**

The Kubernetes implementation is expected to demonstrate horizontal scaling. The final configuration will document:

- Replica counts
- Horizontal Pod Autoscaler configuration
- CPU and memory resource requests/limits
- Scaling commands
- Before/after pod counts
- Validation under increased load

## 18. Monitoring and Logging

### CloudWatch Monitoring

**Status: ⏳ Pending**

The project will document relevant EKS/application metrics, dashboards, alarms, and operational checks.

### Centralized Logging

**Status: ⏳ Pending**

The implementation will centralize application/container logs in AWS and document log groups, retention strategy, and example log queries.

## 19. Security and Configuration Practices

The following practices are used or planned throughout the project:

- Do not commit passwords, access keys, tokens, or database secrets.
- Keep `.env` files outside Docker build contexts through `.dockerignore` rules.
- Supply runtime configuration through environment variables or managed secret mechanisms.
- Use least-privilege IAM permissions for AWS components.
- Use versioned container image tags rather than relying only on `latest`.
- Separate development/local configuration from cloud deployment configuration.

## 20. Troubleshooting

### Docker Engine Unavailable

Confirm Docker Desktop is running and that the Linux container engine is available:

```powershell
docker info
```

### Check Running Containers

```powershell
docker ps
```

### Inspect Container Logs

```powershell
docker logs frontend
docker logs hello-service
docker logs profile-service
docker logs mongodb
```

### Test Service Health

```powershell
curl.exe http://localhost:3001/health
curl.exe http://localhost:3002/health
```

## 21. Evidence and Validation

Project evidence should be captured from the actual implementation rather than recreated from documentation. Recommended evidence includes:

- GitHub commit history
- Docker images and running containers
- Local browser validation
- Amazon ECR repositories and pushed images
- Jenkins successful pipeline executions
- EKS cluster and node status
- Kubernetes pods/services
- Helm release status
- Scaling demonstration
- CloudWatch metrics and logs
- Final end-to-end application access

## 22. Final Deliverable

The completed project will provide a version-controlled, containerized MERN microservices application with a repeatable AWS deployment workflow and operational visibility.

The final README will be updated after each major milestone so that all documented configuration, commands, resource names, and validation results remain aligned with the actual implementation.

---

**Repository:** https://github.com/raviveera2305/SampleMERNwithMicroservices
