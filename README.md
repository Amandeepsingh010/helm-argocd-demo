# GitOps Deployment using K3s, Helm & ArgoCD on AWS EC2 🚀

## Project Overview

This project demonstrates a complete GitOps workflow using:

* K3s (Lightweight Kubernetes)
* Helm
* ArgoCD
* GitHub
* AWS EC2 (Amazon Linux 2023)

The goal of this project is to automate Kubernetes deployments using GitOps principles where GitHub acts as the Single Source of Truth and ArgoCD continuously synchronizes Kubernetes resources automatically.

---

# Architecture

```text
GitHub Repository
        ↓
ArgoCD watches Git repository
        ↓
Helm Chart
        ↓
K3s Kubernetes Cluster
        ↓
NGINX Application
```

---

# Tech Stack

* AWS EC2
* Amazon Linux 2023
* K3s
* Kubernetes
* Helm
* ArgoCD
* GitHub
* GitOps

---

# Prerequisites

Before starting, make sure you have:

* AWS Account / Sandbox Account
* EC2 Key Pair
* GitHub Account
* Internet Access

---

# EC2 Configuration

| Setting       | Value             |
| ------------- | ----------------- |
| AMI           | Amazon Linux 2023 |
| Instance Type | t2.medium         |
| Storage       | 20 GB             |

---

# Security Group Rules

Allow the following inbound ports:

| Type       | Port        |
| ---------- | ----------- |
| SSH        | 22          |
| HTTP       | 80          |
| HTTPS      | 443         |
| Custom TCP | 30000-32767 |

---

# Step 1: Connect to EC2

```bash
ssh -i your-key.pem ec2-user@YOUR_PUBLIC_IP
```

---

# Step 2: Update Server

```bash
sudo dnf update -y
```

---

# Step 3: Install Git

```bash
sudo dnf install git -y
```

Verify installation:

```bash
git --version
```

---

# Step 4: Install K3s

Install lightweight Kubernetes cluster:

```bash
curl -sfL https://get.k3s.io | sh -
```

---

# Step 5: Verify K3s Cluster

```bash
sudo kubectl get nodes
```

Expected Output:

```text
Ready
```

---

# Step 6: Configure kubectl

Export kubeconfig:

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

Make it permanent:

```bash
echo 'export KUBECONFIG=/etc/rancher/k3s/k3s.yaml' >> ~/.bashrc
source ~/.bashrc
```

---

# Step 7: Verify Kubernetes Pods

```bash
kubectl get pods -A
```

---

# Step 8: Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Verify installation:

```bash
helm version
```

---

# Step 9: Install ArgoCD

Create namespace:

```bash
kubectl create namespace argocd
```

Install ArgoCD:

```bash
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

# Step 10: Verify ArgoCD Pods

```bash
kubectl get pods -n argocd
```

Wait until all pods become:

```text
Running
```

---

# Step 11: Expose ArgoCD UI

Convert service to NodePort:

```bash
kubectl patch svc argocd-server -n argocd \
-p '{"spec": {"type": "NodePort"}}'
```

---

# Step 12: Get ArgoCD NodePort

```bash
kubectl get svc -n argocd
```

Example:

```text
443:32000/TCP
```

---

# Step 13: Access ArgoCD UI

Open browser:

```text
https://YOUR_PUBLIC_IP:32000
```

Ignore SSL warning.

---

# Step 14: Get ArgoCD Admin Password

Username:

```text
admin
```

Password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
```

---

# Step 15: Create Project Directory

```bash
mkdir helm-argocd-demo
cd helm-argocd-demo
```

---

# Step 16: Create Helm Chart

```bash
helm create nginx-chart
```

---

# Step 17: Modify values.yaml

Open file:

```bash
vi nginx-chart/values.yaml
```

Update service type:

```yaml
service:
  type: NodePort
  port: 80
```

This allows external browser access to the application.

---

# Step 18: Test Helm Deployment

```bash
helm install nginx-app ./nginx-chart
```

Verify resources:

```bash
kubectl get all
```

---

# Step 19: Access NGINX Application

Get service details:

```bash
kubectl get svc
```

Example:

```text
80:31234/TCP
```

Open browser:

```text
http://YOUR_PUBLIC_IP:31234
```

---

# Step 20: Remove Test Deployment

```bash
helm uninstall nginx-app
```

---

# Step 21: Initialize Git Repository

```bash
git init
```

---

# Step 22: Create application.yaml

Create file:

```bash
vi application.yaml
```

Paste:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application

metadata:
  name: nginx-app
  namespace: argocd

spec:
  project: default

  source:
    repoURL: 'YOUR_GITHUB_REPO_URL'
    targetRevision: HEAD
    path: nginx-chart

  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Replace:

```text
YOUR_GITHUB_REPO_URL
```

with your actual GitHub repository URL.

---

# Step 23: Push Project to GitHub

Add files:

```bash
git add .
```

Commit changes:

```bash
git commit -m "Initial GitOps setup"
```

Rename branch:

```bash
git branch -M main
```

Add remote:

```bash
git remote add origin YOUR_GITHUB_REPO_URL
```

Push code:

```bash
git push -u origin main
```

---

# Step 24: Deploy ArgoCD Application

Apply application manifest:

```bash
kubectl apply -f application.yaml
```

---

# Step 25: Verify ArgoCD Sync

```bash
kubectl get applications -n argocd
```

Expected Output:

```text
Synced
Healthy
```

---

# Step 26: Verify Pods

```bash
kubectl get pods
```

Expected Output:

```text
nginx-app-nginx-chart-xxxxx   Running
```

---

# Step 27: Test GitOps Auto Sync

Edit replicas:

```bash
vi nginx-chart/values.yaml
```

Change:

```yaml
replicaCount: 4
```

Push changes:

```bash
git add .
git commit -m "scaled replicas"
git push
```

ArgoCD automatically synchronizes changes to Kubernetes.

---

# Step 28: Test Self-Healing

Delete deployment manually:

```bash
kubectl delete deployment nginx-app-nginx-chart
```

Watch pods:

```bash
kubectl get pods -w
```

ArgoCD automatically recreates deleted resources.

---

# Project Structure

```text
helm-argocd-demo/
│
├── application.yaml
│
└── nginx-chart/
    │
    ├── Chart.yaml
    ├── values.yaml
    ├── charts/
    └── templates/
         ├── deployment.yaml
         ├── service.yaml
         ├── ingress.yaml
         ├── serviceaccount.yaml
         ├── hpa.yaml
         ├── NOTES.txt
         └── tests/
```

---

# What I Learned

* Kubernetes fundamentals
* Lightweight Kubernetes with K3s
* Helm templating and packaging
* GitOps workflow implementation
* ArgoCD synchronization and self-healing
* Kubernetes networking concepts
* Difference between ClusterIP and NodePort
* Real-world DevOps troubleshooting

---

# GitOps Workflow Explained

Traditional Deployment:

```text
Developer → Manual Deployment → Kubernetes
```

GitOps Deployment:

```text
Developer → GitHub → ArgoCD → Kubernetes
```

ArgoCD continuously watches GitHub and ensures Kubernetes always matches the desired state defined in Git.

---

# Difference Between ClusterIP and NodePort

### ClusterIP

* Internal Kubernetes access only
* Cannot be accessed from browser

### NodePort

* Exposes application externally
* Accessible using:

```text
EC2_PUBLIC_IP:NODEPORT
```

We changed the service type to NodePort to access the application externally through the EC2 public IP.

---

# Conclusion

This project helped me understand how modern DevOps teams automate Kubernetes deployments using GitOps principles, Helm packaging, and ArgoCD synchronization in real-world cloud environments.

---
