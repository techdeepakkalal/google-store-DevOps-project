# Kubernetes + EKS-CTL

This README provides end-to-end steps for installing **eksctl**, **kubectl**, **Docker**, **Ingress-Nginx**, **MariaDB**, **ArgoCD**, and creating necessary namespaces and database tables

---

## Prerequisites

- AWS Account with appropriate IAM permissions
- Linux/Unix environment or WSL2 on Windows
- Sufficient compute resources for EKS cluster

---

## Deployment Workflow

Follow these steps in order for successful deployment:

### Step 1: EKS Cluster Setup with terraform
- Create EKS with terraform file  

### Step 2: EKS Client & Cluster Update
- Already eks clinet server is creted update your terraform keys 
- Update cluster configuration
- Verify cluster connectivity using `kubectl get nodes`

### Step 3: Configure GitHub Secrets
Store the following secrets in your GitHub repository settings:
- AWS Access Key ID
- AWS Secret Access Key
- AWS Account ID
- Git PAT (Personal Access Token)

### Step 4: RDS Database Setup
- Create RDS instance in the same VPC as EKS cluster
- Note the endpoint, username, and password
- Update `backend/app.py` with RDS connection details

### Step 5: Install Dependencies on EKS Server
- Install ArgoCD
- Install Git
- Install MariaDB
- Initialize the database (using SQL commands from Section 6 above)
- k8s ingress controller 

### Step 6: Clone Repository on server
```bash
git clone <your-repository-url>   
```

### Step 7: Deploy Backend
```bash
cd k8s-argocd/backend
kubectl apply -f .
# Wait for Load Balancer to be assigned
kubectl get svc -n google
```

### Step 8: Update Frontend Configuration
- Copy the **Backend Load Balancer URL** from Step 7
- Edit `frontend/main/index.html`
- Update lines 711-712 with the backend URL:
```html
line 711: const API_ENDPOINT = "http://<BACKEND-LB-URL>";
line 712: // Update all API calls to use this endpoint
```

### Step 9: Deploy Frontend
```bash
cd ../frontend
kubectl apply -f .
```

### Step 10: Deploy EFK Stack (Elasticsearch, Fluent Bit, Kibana)
```bash
cd ../efk-stack
kubectl apply -f .
```

### Step 11: Install Grafana & Prometheus
```bash
cd ../../grafana-prometheous
# Follow installation commands in grafana-prometheous/README.md
```

### Step 12: Access the Application
- Get the Ingress Load Balancer URL:
```bash
kubectl get ingress -n google
```
- Access the application using the Ingress Load Balancer URL in your browser

---
## Installation Steps


### 1. Install Git

```bash
yum install git -y
```

### 2. Install Ingress-Nginx

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

### 3. Install MariaDB

```bash
sudo yum update -y
sudo dnf install -y mariadb105
```

### 6. MySQL Database Setup

Connect to MariaDB and execute the following SQL commands:

```sql
CREATE DATABASE IF NOT EXISTS cloud;
USE cloud;
DROP TABLE IF EXISTS users;
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    full_name VARCHAR(150) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
exit
```

### 7. Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl get svc -n argocd
```

### 8. Create Application Namespace

```bash
kubectl create namespace google
```

### 9. Retrieve ArgoCD Initial Admin Password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

---

## 📸 Architecture & Workflow Diagrams

### Workflow Diagram

![Workflow Diagram](https://i.ibb.co/t9WR8c8/drawing.png)

---


## Troubleshooting

### Check Pod Status
```bash
kubectl get pods -n google
kubectl describe pod <pod-name> -n google
kubectl logs <pod-name> -n google
```

### Check Services and Load Balancers
```bash
kubectl get svc -n google
kubectl get ingress -n google
```

### Verify ArgoCD Status
```bash
kubectl get all -n argocd
```

### Check EFK Stack Logs
```bash
kubectl get pods -n kube-logging
kubectl logs <pod-name> -n kube-logging
```


## Summary of Key Components

| Component | Purpose |
|-----------|---------|
| **EKS** | Managed Kubernetes service on AWS |
| **RDS** | Managed relational database (MySQL/MariaDB) |
| **ArgoCD** | Continuous Deployment tool |
| **Ingress-Nginx** | Ingress controller for routing |
| **EFK Stack** | Logging and debugging (Elasticsearch, Fluent Bit, Kibana) |
| **Grafana** | Metrics visualization |
| **Prometheus** | Metrics collection and alerting |

---

**Last Updated:** March 2026
