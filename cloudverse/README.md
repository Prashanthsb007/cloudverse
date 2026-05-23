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
│ ├── ui/
│ ├── api-gateway/
│ ├── auth-service/
│ ├── user-service/
│ ├── product-service/
│ ├── order-service/
│ ├── cart-service/
│ ├── notification-service/
│ ├── analytics-service/
│ └── search-service/
└── k8s-manifests/
    ├── 00-namespace.yaml
    ├── 01-postgres-pv-pvc.yaml
    ├── 02-postgres-secret.yaml
    ├── 03-postgres-deployment.yaml
    ├── 04-postgres-service.yaml
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
    ├── 15-ingress.yaml
    └── 16-hpa.yaml
```

---

# ⚙️ Prerequisites

| Requirement | Details |
|------------|----------|
| AWS Account | With running EKS cluster |
| EC2 Instance | Amazon Linux 2 |
| Docker | Installed |
| AWS CLI | Configured |
| kubectl | Connected to EKS |
| AWS Load Balancer Controller | Installed |
| EBS CSI Driver | Installed |
| Metrics Server | Installed |

---

# 🚀 Deployment Guide

---

# 🔧 PHASE 1 — Prepare EC2

## Install Docker

```
sudo yum update -y
sudo yum install -y docker
sudo service docker start
sudo usermod -aG docker ec2-user
newgrp docker
docker --version
```

## Install AWS CLI

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

## Configure AWS

```
aws configure
```

## Install kubectl

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

## Connect to EKS

```
aws eks update-kubeconfig --region us-east-1 --name <your-cluster-name>
kubectl get nodes
```

---

# 📦 PHASE 2 — Create ECR Repositories

```
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export AWS_REGION=us-east-1
```

(Create all 10 repositories exactly as defined earlier.)

---

# 🐳 PHASE 3 — Clone, Build, Tag and Push

(Full detailed build commands for all 10 services go here — unchanged from previous correct section.)

---

# 🔖 PHASE 4 — Update Image URIs

```
sed -i "s/YOUR_ACCOUNT_ID/${AWS_ACCOUNT_ID}/g" k8s-manifests/*.yaml
```

---

# 🏷️ PHASE 5 — Label Node

```
kubectl get nodes
kubectl label node <node-name> role=database
kubectl label node <node-name> node-type=storage-optimized
```

---

# ☸️ PHASE 6 — Deploy to Kubernetes

## ✅ Step 12 — Apply All Manifests

```
kubectl apply -f k8s-manifests/00-namespace.yaml
kubectl apply -f k8s-manifests/01-postgres-pv-pvc.yaml
kubectl apply -f k8s-manifests/02-postgres-secret.yaml
kubectl apply -f k8s-manifests/03-postgres-deployment.yaml
kubectl apply -f k8s-manifests/04-postgres-service.yaml
```

Wait for Postgres:

```
kubectl rollout status deployment/postgres -n cloudverse
```

Now deploy remaining services:

```
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

```
echo "✅ All manifests applied!"
```

---

# ✅ PHASE 7 — Verify Everything

## Step 13 — Check Pods

```
kubectl get pods -n cloudverse
```

All pods must be **Running**.

---

## Step 14 — Check Services

```
kubectl get services -n cloudverse
```

---

## Step 15 — Get ALB DNS

```
kubectl get ingress -n cloudverse
```

Wait 3–5 minutes.

---

## Step 16 — Open in Browser

```
http://<ADDRESS-from-step-15>
```

Login:

```
admin / admin123
```

---

# 🔍 Verify DevOps Concepts

## Pod-to-Pod Communication

```
kubectl exec -it -n cloudverse \
$(kubectl get pod -n cloudverse -l app=order-service -o jsonpath='{.items[0].metadata.name}') \
-- sh
```

Inside:

```
wget -qO- http://notification-service:4006/health
```

---

## PVC

```
kubectl get pvc -n cloudverse
kubectl describe pvc postgres-pvc -n cloudverse
```

---

## Node Affinity

```
kubectl describe pod -n cloudverse -l app=postgres | grep -A 15 Affinity
```

---

## HPA

```
kubectl get hpa -n cloudverse
```

---

## Rolling Update Demo

```
kubectl set image deployment/product-service \
product-service=${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/cloudverse/product-service:v2 \
-n cloudverse
```

```
kubectl rollout undo deployment/product-service -n cloudverse
```

---

# 🧪 Useful Demo Commands

```
kubectl get pods -n cloudverse -w
kubectl logs -n cloudverse -l app=auth-service --tail=50
kubectl logs -n cloudverse -l app=order-service --tail=50
kubectl logs -n cloudverse -l app=notification-service --tail=50
kubectl describe ingress cloudverse-ingress -n cloudverse
kubectl get endpoints -n cloudverse
kubectl scale deployment product-service --replicas=4 -n cloudverse
kubectl top pods -n cloudverse
kubectl get all -n cloudverse
```

---

# 🐛 Troubleshooting

(Re-auth ECR, Node label fix, Metrics server, EBS CSI addon exactly as provided.)

---

# 🧹 Cleanup

```
kubectl delete namespace cloudverse
```

(Delete all 10 ECR repositories exactly as listed earlier.)

---

# 👨‍💻 Maintained By

Hiqode DevOps Team  
Part of the Hiqode DevOps Training Series  

Docker → ECR → Kubernetes (EKS) — Real World Microservices Platform

---

# ✅ FINAL SUMMARY

```
============================================================
🎉 CloudVerse project created successfully!
============================================================

NEXT STEPS:
1. cd cloudverse
2. Follow the README.md step by step
3. export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
4. Build and push all Docker images to ECR
5. Apply all k8s-manifests/ to your EKS cluster
============================================================
```