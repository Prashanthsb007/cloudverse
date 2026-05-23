# ☁️ CloudVerse — 10-Microservice DevOps Demo on AWS EKS

> A production-grade, visually stunning microservices e-learning platform built to teach DevOps students real-world concepts:
>
> Docker → ECR → Kubernetes (EKS)

---

# 📌 What This Project Demonstrates

| Concept         | How It's Shown                      |
| --------------- | ----------------------------------- |
| Microservices   | 10 independent services             |
| Docker + ECR    | Each service has its own Dockerfile |
| EKS             | All resources deployed on AWS EKS   |
| ALB Ingress     | Single entry point                  |
| HPA             | Auto scaling                        |
| PV/PVC          | PostgreSQL persistence              |
| Node Affinity   | DB scheduled on labeled nodes       |
| Probes          | Liveness + Readiness                |
| Rolling Updates | Zero downtime deployment            |

---

# 🏗️ Architecture

```text
Internet
   │
   ▼
AWS ALB Ingress
   │
   ▼
UI Service (React)
   │
   ▼
API Gateway
   │
   ├── Auth Service
   ├── User Service
   ├── Product Service
   ├── Order Service ──► Notification Service
   ├── Cart Service
   ├── Analytics Service
   └── Search Service

PostgreSQL Database
```

---

# 📁 Project Structure

```text
cloudverse/
├── README.md
├── services/
│   ├── ui/
│   ├── api-gateway/
│   ├── auth-service/
│   ├── user-service/
│   ├── product-service/
│   ├── order-service/
│   ├── cart-service/
│   ├── notification-service/
│   ├── analytics-service/
│   └── search-service/
└── k8s-manifests/
```

---

# ⚙️ Prerequisites

* AWS Account
* EKS Cluster
* Docker
* kubectl
* AWS CLI
* Metrics Server
* AWS Load Balancer Controller
* EBS CSI Driver

---

# 🚀 Deployment Guide

---

# 🔧 Install Docker

```bash
sudo yum update -y
sudo yum install -y docker
sudo service docker start
sudo usermod -aG docker ec2-user
newgrp docker

docker --version
```

---

# 🔧 Install AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

aws --version
```

---

# 🔧 Configure AWS CLI

```bash
aws configure
```

---

# 🔧 Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/

kubectl version --client
```

---

# 🔧 Connect kubectl to EKS

```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name <your-cluster-name>

kubectl get nodes
```

---

# 📦 Create ECR Repositories

```bash
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

export AWS_REGION=us-east-1
```

---

```bash
aws ecr create-repository --repository-name cloudverse/ui-service

aws ecr create-repository --repository-name cloudverse/api-gateway

aws ecr create-repository --repository-name cloudverse/auth-service
```

---

# 🐳 Login Docker to ECR

```bash
aws ecr get-login-password --region us-east-1 \
| docker login --username AWS \
--password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com
```

---

# 🐳 Build UI Docker Image

```bash
docker build -t cloudverse-ui ./services/ui

docker tag cloudverse-ui:latest \
${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/ui-service:v1

docker push \
${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/ui-service:v1
```

---

# 🔖 Update Image URIs

```bash
sed -i "s/YOUR_ACCOUNT_ID/${AWS_ACCOUNT_ID}/g" k8s-manifests/*.yaml
```

---

# 🏷️ Label Database Node

```bash
kubectl get nodes

kubectl label node <node-name> role=database

kubectl label node <node-name> node-type=storage-optimized
```

---

# ☸️ Deploy to Kubernetes

```bash
kubectl apply -f k8s-manifests/00-namespace.yaml

kubectl apply -f k8s-manifests/01-postgres-pv-pvc.yaml

kubectl apply -f k8s-manifests/02-postgres-secret.yaml

kubectl apply -f k8s-manifests/03-postgres-deployment.yaml

kubectl apply -f k8s-manifests/04-postgres-service.yaml
```

---

# 🚀 Deploy All Services

```bash
kubectl apply -f k8s-manifests/05-auth-service.yaml

kubectl apply -f k8s-manifests/06-user-service.yaml

kubectl apply -f k8s-manifests/07-product-service.yaml

kubectl apply -f k8s-manifests/08-order-service.yaml

kubectl apply -f k8s-manifests/09-cart-service.yaml

kubectl apply -f k8s-manifests/10-notification-service.yaml

kubectl apply -f k8s-manifests/11-analytics-service.yaml

kubectl apply -f k8s-manifests/12-search-service.yaml

kubectl apply -f k8s-manifests/13-api-gateway.yaml

kubectl apply -f k8s-manifests/14-ui-service.yaml

kubectl apply -f k8s-manifests/15-ingress.yaml

kubectl apply -f k8s-manifests/16-hpa.yaml
```

---

# ✅ Verify

## Check Pods

```bash
kubectl get pods -n cloudverse
```

---

## Check Services

```bash
kubectl get svc -n cloudverse
```

---

## Check Ingress

```bash
kubectl get ingress -n cloudverse
```

---

# 🧪 Useful Commands

## Watch Pods

```bash
kubectl get pods -n cloudverse -w
```

---

## View Logs

```bash
kubectl logs -n cloudverse -l app=auth-service

kubectl logs -n cloudverse -l app=order-service
```

---

## Scale Deployment

```bash
kubectl scale deployment product-service \
--replicas=4 \
-n cloudverse
```

---

# 🧹 Cleanup

```bash
kubectl delete namespace cloudverse
```

---

# 👨‍💻 Maintained By

Hiqode DevOps Team

Docker → ECR → Kubernetes (EKS)
