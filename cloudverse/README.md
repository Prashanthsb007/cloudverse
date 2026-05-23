# 🌩️ CloudVerse — Microservices Platform on AWS EKS

CloudVerse is a real-world microservices platform deployed on **AWS EKS** using:

- ✅ Docker  
- ✅ Amazon ECR  
- ✅ Kubernetes (EKS)  
- ✅ AWS ALB Ingress  
- ✅ EBS Persistent Volume  
- ✅ HPA (Horizontal Pod Autoscaler)  
- ✅ Node Affinity  
- ✅ Liveness & Readiness Probes  

---

# 📁 Project Structure

```
cloudverse/
├── README.md
├── services/
│ ├── ui/                       ← React frontend
│ ├── api-gateway/              ← Routes all /api/* requests
│ ├── auth-service/             ← Login, Register → PostgreSQL
│ ├── user-service/             ← User profiles → PostgreSQL
│ ├── product-service/          ← Product catalog
│ ├── order-service/            ← Orders + calls notification-service
│ ├── cart-service/             ← Shopping cart (in-memory)
│ ├── notification-service/     ← Receives calls from order-service
│ ├── analytics-service/        ← Platform stats
│ └── search-service/           ← Product search
└── k8s-manifests/
├── 00-namespace.yaml
├── 01-postgres-pv-pvc.yaml           ← PV/PVC (EBS)
├── 02-postgres-secret.yaml           ← DB credentials
├── 03-postgres-deployment.yaml       ← Node Affinity + Probes
├── 04-postgres-service.yaml          ← ClusterIP
├── 05-auth-service.yaml
├── 06-user-service.yaml
├── 07-product-service.yaml
├── 08-order-service.yaml
├── 09-cart-service.yaml
├── 10-notification-service.yaml
├── 11-analytics-service.yaml
├── 12-search-service.yaml
├── 13-api-gateway.yaml
├── 14-ui-service.yaml
├── 15-ingress.yaml                   ← AWS ALB Ingress
└── 16-hpa.yaml                       ← HPA for 4 services
```

---

# ⚙️ Prerequisites

| Requirement | Details |
|------------|----------|
| AWS Account | With running EKS cluster |
| EC2 Instance | Amazon Linux 2 (t2.micro) |
| Docker | Installed |
| AWS CLI | Configured |
| kubectl | Connected to EKS |
| AWS Load Balancer Controller | Installed |
| EBS CSI Driver | Installed |
| Metrics Server | Installed |

---

# 🚀 Step‑By‑Step Deployment Guide

---

# 🔧 PHASE 1 — Prepare EC2 Instance

## Step 1 — Install Docker

```
sudo yum update -y
sudo yum install -y docker
sudo service docker start
sudo usermod -aG docker ec2-user
newgrp docker
docker --version
```

## Step 2 — Install AWS CLI

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

## Step 3 — Configure AWS CLI

```
aws configure
```

## Step 4 — Install kubectl

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

## Step 5 — Connect to EKS

```
aws eks update-kubeconfig --region us-east-1 --name <your-eks-cluster-name>
kubectl get nodes
```

---

# 📦 PHASE 2 — Create ECR Repositories

## Step 6 — Create All 10 Repositories

```
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export AWS_REGION=us-east-1

aws ecr create-repository --repository-name cloudverse/ui-service --region $AWS_REGION
aws ecr create-repository --repository-name cloudverse/api-gateway --region $AWS_REGION
aws ecr create-repository --repository-name cloudverse/auth-service --region $AWS_REGION
aws ecr create-repository --repository-name cloudverse/user-service --region $AWS_REGION
aws ecr create-repository --repository-name cloudverse/product-service --region $AWS_REGION
aws ecr create-repository --repository-name cloudverse/order-service --region $AWS_REGION
aws ecr create-repository --repository-name cloudverse/cart-service --region $AWS_REGION
aws ecr create-repository --repository-name cloudverse/notification-service --region $AWS_REGION
aws ecr create-repository --repository-name cloudverse/analytics-service --region $AWS_REGION
aws ecr create-repository --repository-name cloudverse/search-service --region $AWS_REGION
```

---

# 🐳 PHASE 3 — Clone, Build, Tag and Push

## Step 7 — Clone the Repository

```
git clone https://github.com/<your-org>/cloudverse.git
cd cloudverse
```

## Step 8 — Authenticate Docker with ECR

```
aws ecr get-login-password --region us-east-1 \
| docker login --username AWS \
--password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com
```

---

## Step 9 — Build, Tag and Push All Images

### 🖥️ UI Service

```
docker build -t cloudverse-ui ./services/ui
docker tag cloudverse-ui:latest ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/ui-service:v1
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/ui-service:v1
```

### 🔀 API Gateway

```
docker build -t cloudverse-api-gateway ./services/api-gateway
docker tag cloudverse-api-gateway:latest ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/api-gateway:v1
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/api-gateway:v1
```

### 🔐 Auth Service

```
docker build -t cloudverse-auth-service ./services/auth-service
docker tag cloudverse-auth-service:latest ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/auth-service:v1
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/auth-service:v1
```

### 👤 User Service

```
docker build -t cloudverse-user-service ./services/user-service
docker tag cloudverse-user-service:latest ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/user-service:v1
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/user-service:v1
```

### 🛍️ Product Service

```
docker build -t cloudverse-product-service ./services/product-service
docker tag cloudverse-product-service:latest ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/product-service:v1
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/product-service:v1
```

### 📦 Order Service

```
docker build -t cloudverse-order-service ./services/order-service
docker tag cloudverse-order-service:latest ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/order-service:v1
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/order-service:v1
```

### 🛒 Cart Service

```
docker build -t cloudverse-cart-service ./services/cart-service
docker tag cloudverse-cart-service:latest ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/cart-service:v1
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/cart-service:v1
```

### 🔔 Notification Service

```
docker build -t cloudverse-notification-service ./services/notification-service
docker tag cloudverse-notification-service:latest ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/notification-service:v1
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/notification-service:v1
```

### 📊 Analytics Service

```
docker build -t cloudverse-analytics-service ./services/analytics-service
docker tag cloudverse-analytics-service:latest ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/analytics-service:v1
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/analytics-service:v1
```

### 🔍 Search Service

```
docker build -t cloudverse-search-service ./services/search-service
docker tag cloudverse-search-service:latest ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/search-service:v1
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/search-service:v1
```

---

# 🔖 PHASE 4 — Update Image URIs

```
sed -i "s/YOUR_ACCOUNT_ID/${AWS_ACCOUNT_ID}/g" k8s-manifests/*.yaml
grep "ecr" k8s-manifests/05-auth-service.yaml
```

---

# 🏷️ PHASE 5 — Label Node for Database

```
kubectl get nodes
kubectl label node <node-name> role=database
kubectl label node <node-name> node-type=storage-optimized
kubectl get nodes --show-labels
```

---

# ☸️ PHASE 6 — Deploy to Kubernetes

```
kubectl apply -f k8s-manifests/
kubectl rollout status deployment/postgres -n cloudverse
```

---

# ✅ PHASE 7 — Verify Everything

```
kubectl get pods -n cloudverse
kubectl get services -n cloudverse
kubectl get ingress -n cloudverse
```

Open in browser:

```
http://<ALB-DNS>
```

Login:

```
admin / admin123
```

---

# 🧹 Cleanup

```
kubectl delete namespace cloudverse
```

Delete ECR repositories:

```
aws ecr delete-repository --repository-name cloudverse/ui-service --force --region us-east-1
```

(Repeat for all services.)

---

# 👨‍💻 Maintained By

Hiqode DevOps Team  
Part of the Hiqode DevOps Training Series  

Docker → ECR → Kubernetes (EKS) — Real‑World Microservices Platform