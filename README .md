# MLOps Salary Prediction Pipeline

> **End-to-end MLOps project** — Built in 7 days. From raw CSV to a production-deployed, containerized, CI/CD-automated, monitored ML microservice running on Kubernetes.

[![CI/CD](https://github.com/manu07-oss/mlops-project/actions/workflows/ci-cd.yaml/badge.svg)](https://github.com/manu07-oss/mlops-project/actions)
[![Docker Hub](https://img.shields.io/badge/Docker%20Hub-manognavengala01-blue)](https://hub.docker.com/r/manognavengala01/salary-prediction-api)
[![Python](https://img.shields.io/badge/Python-3.11-green)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.135-teal)](https://fastapi.tiangolo.com)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.35-blue)](https://kubernetes.io)
[![MLflow](https://img.shields.io/badge/MLflow-3.10-orange)](https://mlflow.org)

---

## Project Timeline

| Day | What was built | Time taken |
|-----|---------------|-----------|
| Day 1 | ML model training + MLflow experiment tracking | 3 hours |
| Day 2 | 3-model comparison + best model selection | 2 hours |
| Day 3 | FastAPI REST microservice | 2 hours |
| Day 4 | Docker containerization | 3 hours |
| Day 5 | GitHub Actions CI/CD + Trivy security scanning | 1.5 hours |
| Day 6 | Kubernetes deployment (3 replicas) | 2 hours |
| Day 7 | Prometheus + Grafana monitoring | 1.5 hours |
| **Total** | **Full production MLOps pipeline** | **~15 hours** |

---

## Problem Statement

### What problem does this solve?

In traditional ML workflows:

```
Data Scientist trains model on laptop
        ↓
Hands over a .pkl file to engineering
        ↓
Nobody knows:
  - Which version is in production?
  - What data was it trained on?
  - Is it still accurate?
  - How do we redeploy when it degrades?
  - Who do we call when it breaks at 3am?
```

**This project solves every one of these problems:**

| Problem | Solution |
|---------|---------|
| No experiment history | MLflow tracks every run with params, metrics, artifacts |
| Works on my machine | Docker containerizes everything |
| Manual deployments | GitHub Actions automates build + push on every git push |
| Single point of failure | Kubernetes runs 3 replicas with auto-restart |
| No security checks | Trivy scans code + image for CVEs before deploy |
| No visibility in production | Prometheus + Grafana monitors every request |
| Model only usable by Python devs | FastAPI exposes it as a REST API |

---

## Full System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│  LAYER 1: FRONTEND (current + suggested additions)                      │
│                                                                         │
│  Current:                                                               │
│  └── FastAPI /docs → Swagger UI (interactive API testing)               │
│  └── Grafana :3000 → Real-time monitoring dashboard                     │
│  └── MLflow :5000  → Experiment comparison UI                           │
│                                                                         │
│  Can add:                                                               │
│  └── React App → salary predictor form → calls POST /predict            │
│  └── Streamlit → quick ML demo UI                                       │
│  └── Admin dashboard → model version management                         │
└──────────────────────────────┬──────────────────────────────────────────┘
                               │ HTTP REST
┌──────────────────────────────▼──────────────────────────────────────────┐
│  LAYER 2: API / MICROSERVICE (src/predict.py)                           │
│                                                                         │
│  FastAPI application                                                    │
│  ├── GET  /health   → {"status": "ok", "model": "RandomForest"}         │
│  ├── POST /predict  → {"predicted_salary": 45860.0}                     │
│  ├── GET  /docs     → Swagger UI                                        │
│  └── GET  /metrics  → Prometheus metrics endpoint                       │
│                                                                         │
│  Is this a microservice? YES:                                           │
│  ✅ Single responsibility (salary prediction only)                      │
│  ✅ Independently deployable (own Docker image)                         │
│  ✅ Communicates via REST API (JSON over HTTP)                           │
│  ✅ Stateless (no session, no DB at runtime)                            │
│  ✅ Independently scalable (Kubernetes replicas)                        │
└──────────────────────────────┬──────────────────────────────────────────┘
                               │ loads
┌──────────────────────────────▼──────────────────────────────────────────┐
│  LAYER 3: ML MODEL                                                      │
│                                                                         │
│  Training (offline):                                                    │
│  salary_data.csv → train_v2.py → MLflow tracks 3 models                │
│  → save_model.py → models/best_model.pkl                                │
│                                                                         │
│  Model comparison:                                                      │
│  ├── LinearRegression   MAE: 16,536  R2: 0.61  (baseline)              │
│  ├── RandomForest       MAE: 16,586  R2: 0.75  ✅ WINNER               │
│  └── GradientBoosting   MAE: 16,942  R2: 0.70                          │
│                                                                         │
│  Serving (runtime):                                                     │
│  pickle.load("models/best_model.pkl") → model.predict(input_df)        │
└──────────────────────────────┬──────────────────────────────────────────┘
                               │ packaged into
┌──────────────────────────────▼──────────────────────────────────────────┐
│  LAYER 4: CONTAINERIZATION (Docker)                                     │
│                                                                         │
│  Image: manognavengala01/salary-prediction-api:latest                   │
│  Base:  python:3.11-slim                                                │
│  Size:  673MB                                                           │
│  Port:  8000                                                            │
└──────────────────────────────┬──────────────────────────────────────────┘
                               │ automated by
┌──────────────────────────────▼──────────────────────────────────────────┐
│  LAYER 5: CI/CD (GitHub Actions)                                        │
│                                                                         │
│  git push → Trivy FS scan → docker build → Trivy image scan             │
│           → push to Docker Hub                                          │
└──────────────────────────────┬──────────────────────────────────────────┘
                               │ deployed to
┌──────────────────────────────▼──────────────────────────────────────────┐
│  LAYER 6: ORCHESTRATION (Kubernetes / Minikube)                         │
│                                                                         │
│  3 replicas | LoadBalancer | liveness + readiness probes                │
│  Rolling updates | Resource limits | Auto-restart                       │
└──────────────────────────────┬──────────────────────────────────────────┘
                               │ monitored by
┌──────────────────────────────▼──────────────────────────────────────────┐
│  LAYER 7: MONITORING                                                    │
│                                                                         │
│  Prometheus :9090 → scrapes /metrics every 15s                          │
│  Grafana :3000    → visualizes request rate, latency, errors            │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Tech Stack

| Tool | Version | Layer | Why this tool |
|------|---------|-------|--------------|
| Python | 3.11 | Core | ML + API language |
| scikit-learn | latest | ML | Industry standard ML library |
| MLflow | 3.10 | Experiment tracking | Like Git for ML experiments |
| FastAPI | 0.135 | API | Auto-docs, type validation, async |
| Pydantic | 2.x | Validation | Input/output schema enforcement |
| Uvicorn | latest | ASGI server | Runs FastAPI in production |
| Docker | latest | Containerization | Portable, reproducible deployment |
| GitHub Actions | latest | CI/CD | Automated pipeline on every push |
| Trivy | latest | Security | CVE scanning for code + images |
| Kubernetes | v1.35 | Orchestration | Scale, self-heal, rolling updates |
| Minikube | v1.38 | Local K8s | Run Kubernetes on laptop |
| Prometheus | latest | Metrics | Scrape + store time-series metrics |
| Grafana | latest | Visualization | Dashboard for metrics |
| prometheus-fastapi-instrumentator | 7.1.0 | Metrics bridge | Expose FastAPI metrics to Prometheus |

---

## Project Structure

```
mlops-project/
│
├── data/
│   └── salary_data.csv              # 15-row training dataset
│
├── src/
│   ├── __init__.py                  # makes src a Python package
│   ├── train.py                     # Day 1: baseline LinearRegression
│   ├── train_v2.py                  # Day 2: 3-model comparison
│   ├── save_model.py                # exports best model to pickle
│   └── predict.py                   # FastAPI microservice + /metrics
│
├── models/
│   └── best_model.pkl               # exported RandomForest model
│
├── monitoring/
│   └── prometheus.yml               # Prometheus scrape config
│
├── k8s/
│   ├── deployment.yaml              # 3 replicas, probes, resource limits
│   └── service.yaml                 # LoadBalancer on port 80
│
├── .github/
│   └── workflows/
│       └── ci-cd.yaml               # Trivy + Docker build + push pipeline
│
├── docker-compose.yml               # Prometheus + Grafana stack
├── Dockerfile                       # python:3.11-slim, port 8000
├── requirements.txt                 # pinned Linux-compatible dependencies
├── mlflow.db                        # local MLflow SQLite tracking store
└── README.md
```

---

## All Commands Used — Morning to End

### Environment Setup

```powershell
# Create project structure
mkdir mlops-project
cd mlops-project
mkdir data notebooks src

# Create virtual environment
python -m venv venv

# Activate venv (Git Bash)
source venv/Scripts/activate

# Activate venv (PowerShell)
venv\Scripts\activate

# Install packages
pip install pandas scikit-learn mlflow xgboost fastapi uvicorn
pip install prometheus-fastapi-instrumentator

# Verify all installed
python -c "import pandas; import sklearn; import mlflow; import xgboost; print('All good!')"
```

### Training

```powershell
# Train baseline model
python src/train.py

# Train and compare 3 models
python src/train_v2.py

# Export best model to pickle
python src/save_model.py

# View MLflow UI
mlflow ui
# open http://localhost:5000
```

### API

```powershell
# Run API locally
python -m uvicorn src.predict:app --reload

# Test health
curl http://localhost:8000/health

# Test prediction
curl -X POST http://localhost:8000/predict \
  -H "Content-Type: application/json" \
  -d '{"years_experience": 5, "education_level": 3}'

# View metrics
curl http://localhost:8000/metrics

# View docs
# open http://localhost:8000/docs
```

### Docker

```powershell
# Build image
docker build -t salary-prediction-api:v1 .

# Build without cache (when code changes not picked up)
docker build --no-cache -t salary-prediction-api:v5 .

# Run container
docker run -p 8000:8000 salary-prediction-api:v1

# Run on different port (if 8000 busy)
docker run -p 9000:8000 salary-prediction-api:v1

# List images
docker images

# List running containers
docker ps

# Stop all containers
docker stop $(docker ps -q)

# View what's inside the container
docker run salary-prediction-api:v1 cat /app/src/predict.py

# Pull from Docker Hub
docker pull manognavengala01/salary-prediction-api:latest
```

### Git

```powershell
# Initialize repo
git init

# Stage all files
git add .

# Commit
git commit -m "Day 1: salary prediction model with MLflow tracking"

# Add remote
git remote add origin https://github.com/manu07-oss/mlops-project.git

# Push (first time)
git push -u origin main

# Force push (when remote has conflicts)
git push -u origin main --force

# Check status
git status
```

### Kubernetes

```powershell
# Start minikube
minikube start

# Check minikube version
minikube version

# Apply deployment
kubectl apply -f k8s/deployment.yaml

# Apply service
kubectl apply -f k8s/service.yaml

# Check pods
kubectl get pods

# Watch pods live
kubectl get pods -w

# Describe pod (debug)
kubectl describe pod <pod-name>

# Get service URL
minikube service salary-prediction-service --url

# Open Kubernetes dashboard
minikube dashboard

# Get all services
kubectl get services

# Check logs of a pod
kubectl logs <pod-name>
```

### Monitoring

```powershell
# Start Prometheus + Grafana
docker-compose up -d

# Check running containers
docker-compose ps

# Stop monitoring stack
docker-compose down

# View Prometheus
# open http://localhost:9090

# View Grafana
# open http://localhost:3000
# login: admin / admin
```

---

## Troubleshooting Commands + Real Company Scenarios

### 1. pywin32 on Linux (Docker build failure)

**Error:**
```
ERROR: Could not find a version that satisfies the requirement pywin32==311
```

**Why it happens:**
`pip freeze` on Windows captures ALL installed packages including Windows-only ones like `pywin32`. Docker runs Linux — Linux has no concept of `pywin32`. This happens in every company when developers build on Windows and deploy to Linux.

**Real company scenario:**
> A DevOps engineer at a fintech company ran `pip freeze > requirements.txt` on their Windows laptop. The Docker build failed in CI/CD because the GitHub Actions runner (Ubuntu) couldn't install `pywin32`. The fix took 2 hours of debugging.

**Fix:**
```powershell
# Remove Windows-only packages
Get-Content requirements.txt | Where-Object { $_ -notmatch "pywin32|colorama" } | Set-Content requirements_new.txt
Remove-Item requirements.txt
Rename-Item requirements_new.txt requirements.txt
```

---

### 2. MLflow Windows path in Docker (OSError at runtime)

**Error:**
```
OSError: No such file or directory: '/tmp/tmpxxx/model/MLmodel'
INFO mlflow: No artifacts found at file:D:/devops-projects/mlops-project/mlruns/...
```

**Why it happens:**
MLflow stores model artifact paths as absolute paths in its SQLite database at training time. When you train on Windows, it saves `D:/devops-projects/...` inside `mlflow.db`. The Docker container runs on Linux and tries to load from that Windows path — which doesn't exist.

**Real company scenario:**
> An ML engineer at a startup trained models locally and committed `mlflow.db` to the repo. The Kubernetes pod crashed on startup because it tried to load model files from `C:/Users/engineer/...`. Production was down for 3 hours.

**Fix:**
```python
# src/save_model.py — export model to portable file
import pickle
model = mlflow.sklearn.load_model(f"runs:/{best_run_id}/model")
with open("models/best_model.pkl", "wb") as f:
    pickle.dump(model, f)

# src/predict.py — load from file, not MLflow
with open("models/best_model.pkl", "rb") as f:
    model = pickle.load(f)
```

---

### 3. Kubernetes liveness probe CrashLoopBackOff

**Error:**
```
Warning  Unhealthy  kubelet  Liveness probe failed: connection refused
Normal   Killing    kubelet  Container failed liveness probe, will be restarted
```

**Why it happens:**
`initialDelaySeconds: 10` means Kubernetes checks `/health` after 10 seconds. Loading a scikit-learn RandomForest model from disk takes longer than 10 seconds on a resource-constrained node. Kubernetes thinks the pod is dead and kills it — restart loop begins.

**Real company scenario:**
> A platform team at an e-commerce company deployed a new recommendation model. The model was 500MB and took 45 seconds to load. The default liveness probe (10s delay) caused all pods to crash loop. Traffic was routed to 0 healthy pods. Fix: increase `initialDelaySeconds` to 90.

**Fix:**
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8000
  initialDelaySeconds: 60   # was 10, now 60
  periodSeconds: 15

readinessProbe:
  httpGet:
    path: /health
    port: 8000
  initialDelaySeconds: 45   # was 10, now 45
  periodSeconds: 10
```

**Debug commands:**
```powershell
kubectl describe pod <pod-name>    # see Events section
kubectl logs <pod-name>            # see what the app printed
kubectl get pods -w                # watch status change live
```

---

### 4. Docker layer cache (stale code)

**Error:**
```
CACHED [5/5] COPY . .    ← old code copied!
```

**Why it happens:**
Docker caches every layer. If nothing changed in the layer inputs, it reuses the cache. When you edit `predict.py`, Docker sometimes uses the cached `COPY . .` layer and packages the OLD file.

**Real company scenario:**
> A developer at a bank pushed a security fix to their API. The CI/CD pipeline showed green but the container still had the old vulnerable code because the Docker cache wasn't invalidated. The fix only reached production after manually clearing the cache.

**Fix:**
```powershell
docker build --no-cache -t salary-prediction-api:v5 .
```

---

### 5. venv broken after moving folder

**Error:**
```
Fatal error in launcher: Unable to create process using '"D:\old-path\venv\Scripts\python.exe"'
```

**Why it happens:**
Python virtual environments store absolute paths internally. When you move the project folder, the venv still points to the old location.

**Fix:**
```powershell
Remove-Item -Recurse -Force venv
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

---

### 6. GitHub Actions secrets not found

**Error:**
```
Username and password required
Error: Username and password required
```

**Why it happens:**
Secrets are repo-specific. When you moved the project to a new repo (`mlops-project`), the secrets stayed in the old repo (`devops-ai-projects`).

**Fix:**
```
GitHub repo → Settings → Secrets and variables → Actions → New repository secret
DOCKER_USERNAME = manognavengala01
DOCKER_PASSWORD = <docker hub access token>
```

---

### 7. PowerShell mkdir multiple arguments

**Error:**
```
mkdir: A positional parameter cannot be found that accepts argument 'notebooks'
```

**Why it happens:**
PowerShell `mkdir` doesn't accept multiple arguments like Linux `mkdir`.

**Fix:**
```powershell
# Option 1: one by one
mkdir data
mkdir notebooks
mkdir src

# Option 2: PowerShell native
New-Item -ItemType Directory -Name data, notebooks, src
```

---

## SAD — Solution Architecture Document

### Document Information

| Field | Value |
|-------|-------|
| Project | MLOps Salary Prediction Pipeline |
| Author | Manogna |
| Version | 1.0 |
| Date | March 2026 |
| Status | Production (Minikube) |

### Executive Summary

This document describes the architecture of an end-to-end MLOps pipeline that operationalizes a machine learning salary prediction model. The system demonstrates production ML engineering practices including experiment tracking, model versioning, REST API serving, CI/CD automation, container orchestration, and observability.

### Business Requirements

| Requirement | Priority | Solution |
|------------|---------|---------|
| Reproducible model training | High | MLflow experiment tracking |
| Model comparison and selection | High | MLflow parallel coordinates UI |
| API access for downstream systems | High | FastAPI REST microservice |
| Zero-downtime deployments | High | Kubernetes rolling updates |
| Automated build pipeline | Medium | GitHub Actions |
| Security scanning | Medium | Trivy CVE scanning |
| Production monitoring | Medium | Prometheus + Grafana |

### Non-Functional Requirements

| NFR | Target | Solution |
|-----|--------|---------|
| Availability | 99.9% | 3 Kubernetes replicas |
| Latency | < 200ms per prediction | FastAPI async + pickle model |
| Scalability | Horizontal | Kubernetes HPA (future) |
| Security | No critical CVEs | Trivy pipeline gate |
| Observability | Full request visibility | Prometheus + Grafana |

### Component Design

**Training Pipeline (offline)**
- Input: `data/salary_data.csv`
- Process: `src/train_v2.py` trains 3 models, logs to MLflow
- Output: best model exported to `models/best_model.pkl`
- Trigger: manual (future: scheduled retraining with Argo Workflows)

**Serving Layer (online)**
- Framework: FastAPI with Uvicorn ASGI server
- Model loading: pickle at startup, held in memory
- Input validation: Pydantic schemas
- Endpoints: `/health`, `/predict`, `/metrics`, `/docs`
- Startup time: ~30-60 seconds (model loading)

**CI/CD Pipeline**
- Trigger: push to main branch
- Steps: code scan → build → image scan → push
- Security gate: Trivy blocks on CRITICAL/HIGH CVEs (exit-code: 0 currently, set to 1 for hard gate)
- Registry: Docker Hub (public)

**Orchestration**
- Platform: Kubernetes (Minikube local, EKS/AKS for production)
- Replicas: 3
- Strategy: RollingUpdate (maxSurge: 1, maxUnavailable: 0)
- Health: liveness (60s delay) + readiness (45s delay)
- Resources: 256Mi-512Mi RAM, 250m-500m CPU

**Observability**
- Metrics: Prometheus scrapes `/metrics` every 15s
- Dashboards: Grafana visualizes request rate, latency, error rate
- Alerts: (future) alert rules for error rate > 1% or latency > 500ms

### Data Flow

```
User/Client
    │
    │ POST /predict {"years_experience": 5, "education_level": 3}
    ▼
Kubernetes LoadBalancer Service (port 80)
    │
    │ routes to one of 3 pods
    ▼
FastAPI Pod (port 8000)
    │
    │ validates input with Pydantic
    │ creates pandas DataFrame
    │ calls model.predict()
    ▼
RandomForest Model (in memory)
    │
    │ returns prediction
    ▼
FastAPI Response
    │
    │ {"predicted_salary": 45860.0, "model_used": "RandomForest"}
    ▼
User/Client

Prometheus (async, every 15s)
    │ scrapes /metrics
    ▼
Grafana dashboard updates
```

### Security Architecture

| Layer | Control | Tool |
|-------|---------|------|
| Code | Static analysis | Trivy filesystem scan |
| Container | CVE scanning | Trivy image scan |
| Secrets | Not hardcoded | GitHub Actions secrets |
| Network | Pod isolation | Kubernetes NetworkPolicy (future) |
| Auth | Not implemented | API Gateway + JWT (future) |

### Known Limitations

| Limitation | Impact | Future fix |
|-----------|--------|-----------|
| Small training dataset (15 rows) | Low model accuracy | Real salary dataset |
| No authentication on API | Security risk in production | API Gateway + JWT |
| Model drift not detected | Accuracy degrades silently | Evidently AI |
| Manual model retraining | Stale model over time | Argo Workflows |
| Local storage (pickle file) | Not scalable | S3/Azure Blob model registry |

---

## AWS EKS Deployment Guide

### Architecture on AWS

```
GitHub (code)
    ↓ git push
GitHub Actions
    ↓ docker build + push
Amazon ECR (Elastic Container Registry)
    ↓ kubectl apply
Amazon EKS (Elastic Kubernetes Service)
    ├── Node Group (EC2 instances)
    │   ├── Pod 1: salary-prediction-api
    │   ├── Pod 2: salary-prediction-api
    │   └── Pod 3: salary-prediction-api
    └── AWS Load Balancer (ALB)
          ↓
       Users / clients
```

### Required AWS Services

| Service | Purpose | Required |
|---------|---------|---------|
| EKS | Kubernetes control plane | Yes |
| EC2 | Worker nodes | Yes |
| ECR | Docker image registry | Yes |
| IAM | Roles and permissions | Yes |
| VPC | Network isolation | Yes |
| Security Groups | Firewall rules | Yes |
| ALB | Load balancer | Yes |
| S3 | Model storage (future) | Optional |
| CloudWatch | Logs + metrics | Optional |

### Step-by-Step EKS Deployment with Terraform

**Step 1: Prerequisites**
```bash
# Install tools
aws configure                    # set AWS_ACCESS_KEY_ID, SECRET, REGION
terraform --version              # 1.5+
kubectl version
eksctl version
```

**Step 2: Terraform — VPC + EKS**

Create `terraform/main.tf`:
```hcl
provider "aws" {
  region = "ap-south-1"   # Mumbai — closest to Hyderabad
}

# VPC
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "mlops-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["ap-south-1a", "ap-south-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true

  tags = {
    "kubernetes.io/cluster/mlops-cluster" = "shared"
  }
}

# EKS Cluster
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "20.0.0"

  cluster_name    = "mlops-cluster"
  cluster_version = "1.29"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    mlops-nodes = {
      min_size     = 1
      max_size     = 3
      desired_size = 2

      instance_types = ["t3.medium"]
    }
  }
}

# ECR Repository
resource "aws_ecr_repository" "salary_api" {
  name                 = "salary-prediction-api"
  image_tag_mutability = "MUTABLE"

  image_scanning_configuration {
    scan_on_push = true
  }
}
```

**Step 3: IAM for GitHub Actions**
```hcl
# IAM role for GitHub Actions to push to ECR
resource "aws_iam_user" "github_actions" {
  name = "github-actions-mlops"
}

resource "aws_iam_user_policy" "ecr_push" {
  name = "ecr-push-policy"
  user = aws_iam_user.github_actions.name

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "ecr:GetAuthorizationToken",
          "ecr:BatchCheckLayerAvailability",
          "ecr:PutImage",
          "ecr:InitiateLayerUpload",
          "ecr:UploadLayerPart",
          "ecr:CompleteLayerUpload"
        ]
        Resource = "*"
      }
    ]
  })
}
```

**Step 4: Security Groups**
```hcl
# Security group for EKS nodes
resource "aws_security_group" "eks_nodes" {
  name   = "mlops-eks-nodes-sg"
  vpc_id = module.vpc.vpc_id

  ingress {
    from_port   = 8000
    to_port     = 8000
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]   # internal only
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]    # public load balancer
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

**Step 5: Deploy Infrastructure**
```bash
cd terraform
terraform init
terraform plan
terraform apply
```

**Step 6: Update GitHub Actions for ECR**

Update `.github/workflows/ci-cd.yaml`:
```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ap-south-1

- name: Login to Amazon ECR
  id: login-ecr
  uses: aws-actions/amazon-ecr-login@v2

- name: Build and push to ECR
  env:
    ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
    ECR_REPOSITORY: salary-prediction-api
  run: |
    docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
    docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest
```

**Step 7: Update Kubernetes to pull from ECR**
```yaml
# k8s/deployment.yaml
spec:
  containers:
  - name: salary-prediction-api
    image: <account-id>.dkr.ecr.ap-south-1.amazonaws.com/salary-prediction-api:latest
```

**Step 8: Connect kubectl to EKS**
```bash
aws eks update-kubeconfig --name mlops-cluster --region ap-south-1
kubectl get nodes                    # verify connected
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get svc                      # get AWS ALB URL
```

---

## Azure AKS Deployment Guide

### Architecture on Azure

```
GitHub (code)
    ↓ git push
GitHub Actions
    ↓ docker build + push
Azure Container Registry (ACR)
    ↓ kubectl apply
Azure Kubernetes Service (AKS)
    ├── Node Pool (Azure VMs)
    │   ├── Pod 1: salary-prediction-api
    │   ├── Pod 2: salary-prediction-api
    │   └── Pod 3: salary-prediction-api
    └── Azure Load Balancer
          ↓
       Users / clients
```

### Step-by-Step AKS Deployment with Terraform

**Step 1: Terraform — ACR + AKS**

```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "mlops" {
  name     = "mlops-rg"
  location = "Central India"
}

# Azure Container Registry
resource "azurerm_container_registry" "acr" {
  name                = "mlopsacr"
  resource_group_name = azurerm_resource_group.mlops.name
  location            = azurerm_resource_group.mlops.location
  sku                 = "Basic"
  admin_enabled       = true
}

# AKS Cluster
resource "azurerm_kubernetes_cluster" "aks" {
  name                = "mlops-aks"
  location            = azurerm_resource_group.mlops.location
  resource_group_name = azurerm_resource_group.mlops.name
  dns_prefix          = "mlops"

  default_node_pool {
    name       = "default"
    node_count = 2
    vm_size    = "Standard_D2_v2"
  }

  identity {
    type = "SystemAssigned"
  }
}

# Grant AKS permission to pull from ACR
resource "azurerm_role_assignment" "aks_acr" {
  principal_id                     = azurerm_kubernetes_cluster.aks.kubelet_identity[0].object_id
  role_definition_name             = "AcrPull"
  scope                            = azurerm_container_registry.acr.id
  skip_service_principal_aad_check = true
}
```

**Step 2: Update GitHub Actions for ACR**
```yaml
- name: Login to ACR
  uses: azure/docker-login@v1
  with:
    login-server: mlopsacr.azurecr.io
    username: ${{ secrets.ACR_USERNAME }}
    password: ${{ secrets.ACR_PASSWORD }}

- name: Build and push to ACR
  run: |
    docker build -t mlopsacr.azurecr.io/salary-prediction-api:latest .
    docker push mlopsacr.azurecr.io/salary-prediction-api:latest
```

**Step 3: Connect kubectl to AKS**
```bash
az aks get-credentials --resource-group mlops-rg --name mlops-aks
kubectl get nodes
kubectl apply -f k8s/
kubectl get svc    # get Azure Load Balancer IP
```

---

## Interview Answers

### "Tell me about this MLOps project"

> "I built an end-to-end MLOps pipeline from scratch in 7 days. Starting with a salary prediction problem, I trained and compared three models — LinearRegression, RandomForest, and GradientBoosting — using MLflow to track every experiment. RandomForest won with R2 of 0.75. I wrapped it as a FastAPI REST microservice, containerized it with Docker, automated the build and security scanning pipeline with GitHub Actions and Trivy, deployed on Kubernetes with 3 replicas and health probes, and added Prometheus and Grafana for production monitoring. The whole pipeline — from git push to a monitored, running container — is fully automated."

### "What is MLflow and how did you use it?"

> "MLflow is an open-source platform for managing the ML lifecycle. I used it to solve a core problem — when you train 10 models, how do you remember which parameters gave which results? MLflow logs every run automatically with a unique run ID, parameters, metrics like MAE and R2, and the model artifact. It also provides a comparison UI with parallel coordinates plots so you can pick the best model based on data. Think of it like Git, but for ML experiments."

### "What is the difference between Docker image and container?"

> "A Docker image is like a recipe — a frozen snapshot of your application and everything it needs to run. It doesn't execute anything by itself. A container is a running instance of that image — the actual live process. You can run multiple containers from the same image, which is exactly what Kubernetes does with replicas. In my project, I built an image with docker build and ran it with docker run, then Kubernetes managed 3 containers from that same image."

### "Why Kubernetes over just Docker?"

> "Docker runs a single container on a single machine. Kubernetes manages many containers across many machines. I needed three things Docker alone can't give me: automatic restart when a pod crashes at 3am, zero-downtime rolling updates when I push a new model version, and horizontal scaling when request load increases. I also configured liveness and readiness probes so Kubernetes knows when a pod is healthy — which was critical because my ML model takes 60 seconds to load."

### "What is Trivy and why two scans?"

> "Trivy is an open-source security scanner from Aqua Security. I run it twice in my pipeline. First, I scan the filesystem before building — this catches known CVEs in my Python packages from requirements.txt. Second, I scan the built Docker image — this catches vulnerabilities introduced by the base image like python:3.11-slim. Two scans because vulnerabilities can enter at two different points: your code dependencies and the OS layer of the container."

### "Walk me through a prediction request end to end"

> "A POST request hits the Kubernetes LoadBalancer service on port 80. It routes to one of 3 healthy pods — determined by the readiness probe. Inside the pod, FastAPI receives the JSON body, Pydantic validates the types (years_experience as float, education_level as int), we create a pandas DataFrame, and call model.predict(). The RandomForest model — loaded into memory at pod startup from a pickle file — returns the prediction. FastAPI serializes it as JSON and sends the response. Prometheus captures the request duration and increments the counter. Total time: under 100ms."

### "What bugs did you hit and how did you fix them?"

> "Three interesting ones. First, pywin32 — pip freeze on Windows captures OS-specific packages. Docker runs Linux and can't install pywin32. Fixed by filtering requirements.txt. Second, MLflow path issue — MLflow hardcodes Windows paths like D:/projects/mlruns into its SQLite database. The container looked for those paths on Linux and crashed. Fixed by exporting the model to a portable pickle file at build time. Third, Kubernetes liveness probe — the probe checked /health after 10 seconds but loading the ML model took longer. Kubernetes thought the pod was dead and restarted it in a loop. Fixed by increasing initialDelaySeconds to 60."

### "How would you deploy this to AWS in production?"

> "I'd use Terraform to provision an EKS cluster in a private VPC with two availability zones. Worker nodes would be in private subnets with a NAT gateway for outbound traffic. I'd replace Docker Hub with ECR — Amazon's private container registry — and add an IAM role for GitHub Actions to push images. The Kubernetes service would use an AWS Load Balancer Controller to create an ALB instead of the Minikube LoadBalancer. For monitoring I'd add CloudWatch Container Insights alongside Prometheus. For secrets I'd use AWS Secrets Manager instead of Kubernetes secrets."

---

## What is pending

- [ ] Evidently AI — model drift detection
- [ ] Argo Workflows — automated model retraining pipeline
- [ ] React frontend — salary predictor UI
- [ ] AWS EKS — production cloud deployment
- [ ] Horizontal Pod Autoscaler — auto-scale based on CPU/memory
- [ ] API authentication — JWT tokens
- [ ] Alerting rules — Prometheus alerts to Slack/PagerDuty
- [ ] Model registry — promote models through staging → production
- [ ] Larger dataset — improve model accuracy beyond R2: 0.75

---

## How to run the full stack locally

```bash
# Terminal 1: API
python -m uvicorn src.predict:app --reload

# Terminal 2: Monitoring
docker-compose up -d

# Terminal 3: Kubernetes
minikube start
kubectl apply -f k8s/
minikube service salary-prediction-service --url

# Terminal 4: MLflow
mlflow ui

# URLs:
# API:        http://localhost:8000/docs
# MLflow:     http://localhost:5000
# Prometheus: http://localhost:9090
# Grafana:    http://localhost:3000 (admin/admin)
# K8s:        <minikube url>/docs
```

---

## Author

**Manogna** — DevOps Engineer (2.7 years) → MLOps / AI Infrastructure Engineer

Core stack: AWS, Azure, Terraform, Kubernetes (EKS), GitLab CI/CD, GitHub Actions, Docker, Helm, Datadog, Prometheus, Grafana

This project: MLflow, FastAPI, scikit-learn, Trivy, Minikube, Docker Compose

GitHub: [manu07-oss](https://github.com/manu07-oss)
Docker Hub: [manognavengala01](https://hub.docker.com/u/manognavengala01)

> "DevOps engineers who understand ML infrastructure are rare. This project proves I can operate at both layers."