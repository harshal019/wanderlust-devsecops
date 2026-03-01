Perfect 👌
Now I will give you a **complete, polished, production-ready documentation (FINAL VERSION)** including:

* Architecture
* CI/CD
* Security
* Monitoring
* OIDC
* ALB
* SSL
* Versioning
* Repo structure
* Verification commands
* Cleanup
* Learning outcomes
* Future improvements

You can directly paste this into:

```
DOCUMENTATION.md
```

or use inside `README.md`.

---

# 🚀 Wanderlust – Production-Grade DevSecOps Implementation

---

# 📌 Project Overview

Wanderlust is a cloud-native MERN stack application deployed on Amazon EKS using a complete DevSecOps and GitOps workflow.

This project demonstrates:

* CI/CD pipeline automation
* Security scanning integration
* Kubernetes-based container orchestration
* AWS ALB Ingress with SSL
* GitOps deployment model using ArgoCD
* Monitoring using Prometheus & Grafana

This is a production-style deployment implementation.

---

# 🏗️ High-Level Architecture

## 🔹 Application Traffic Flow

```
User
  ↓
Route53 (DNS)
  ↓
AWS Application Load Balancer (HTTPS 443)
  ↓
Kubernetes Ingress
  ↓
Frontend Service (ClusterIP)
  ↓
Backend Service (ClusterIP)
  ↓
MongoDB + Redis
```

---

## 🔹 CI/CD Flow

```
Developer
  ↓
GitHub (Source Code)
  ↓
Jenkins CI Pipeline
  ↓
DockerHub (Image Registry)
  ↓
GitOps Repository (Manifest Update)
  ↓
ArgoCD (Auto Sync)
  ↓
Amazon EKS Cluster
```

---

# 🧱 Tech Stack

| Layer            | Tool                   |
| ---------------- | ---------------------- |
| Source Control   | GitHub                 |
| CI               | Jenkins                |
| Code Quality     | SonarQube              |
| Dependency Scan  | OWASP Dependency Check |
| Image Scan       | Trivy                  |
| Containerization | Docker                 |
| Registry         | DockerHub              |
| CD               | ArgoCD                 |
| Orchestration    | Amazon EKS             |
| Load Balancer    | AWS ALB                |
| DNS              | Route53                |
| SSL              | AWS ACM                |
| Monitoring       | Prometheus + Grafana   |
| Caching          | Redis                  |
| Database         | MongoDB                |

---

# ⚙️ PHASE 1 – Infrastructure Setup

---

## 1️⃣ Create EKS Cluster

```bash
eksctl create cluster \
  --name wanderlust \
  --region us-east-2 \
  --version 1.29 \
  --without-nodegroup
```

---

## 2️⃣ Associate IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --region us-east-2 \
  --cluster wanderlust \
  --approve
```

This is required for AWS Load Balancer Controller.

---

## 3️⃣ Create Nodegroup

```bash
eksctl create nodegroup \
  --cluster=wanderlust \
  --region=us-east-2 \
  --name=wanderlust-ng \
  --node-type=t2.large \
  --nodes=2 \
  --nodes-min=2 \
  --nodes-max=2 \
  --managed
```

---

# ☁️ AWS Load Balancer Controller Setup

---

## Create IAM Policy

```bash
curl -o iam_policy.json \
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

---

## Create IAM Service Account

```bash
eksctl create iamserviceaccount \
  --cluster wanderlust \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

---

## Install Controller

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=wanderlust \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

---

# 🔁 PHASE 2 – CI Pipeline (Jenkins)

---

## CI Stages

1. Git Checkout
2. SonarQube Analysis
3. Quality Gate Validation
4. OWASP Dependency Scan
5. Trivy File Scan
6. Docker Image Build
7. Trivy Image Scan
8. Docker Push
9. Trigger CD Pipeline

---

## 📦 Image Versioning Strategy

Images tagged using:

```
wanderlust-api:v${BUILD_NUMBER}
wanderlust-web:v${BUILD_NUMBER}
```

Benefits:

* Rollback capability
* Traceability
* Version control

---

# 🔄 PHASE 3 – GitOps CD (ArgoCD)

---

## Install ArgoCD

```bash
kubectl create namespace argocd

kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

## Add Cluster to ArgoCD

```bash
argocd login <ARGO_SERVER>

argocd cluster list

kubectl config get-contexts

argocd cluster add <CLUSTER_CONTEXT>
```

---

## GitOps Deployment Flow

* Jenkins updates image tag
* Push to GitOps repo
* ArgoCD auto-sync
* EKS updated automatically

---

# 🌐 Ingress + SSL + Domain Setup

---

## ACM Certificate

Request certificate in AWS ACM:

Domain:

```
wanderlust.harshalgharat.site
```

---

## Ingress Configuration

```yaml
annotations:
  kubernetes.io/ingress.class: alb
  alb.ingress.kubernetes.io/scheme: internet-facing
  alb.ingress.kubernetes.io/target-type: ip
  alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
  alb.ingress.kubernetes.io/certificate-arn: <ACM_ARN>
```

Routing:

* `/` → frontend-svc
* `/api` → backend-svc

---

## Route53 Mapping

Create A record:

* Name: wanderlust.harshalgharat.site
* Type: A
* Alias → ALB DNS

No separate `/api` record required.

---

# 📊 Monitoring Setup (Prometheus + Grafana)

---

## Install Helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

---

## Install Monitoring Stack

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

kubectl create namespace monitoring

helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring
```

---

## Access Grafana

```bash
kubectl edit svc monitoring-grafana -n monitoring
```

Change:

```
type: NodePort
```

Get password:

```bash
kubectl get secret monitoring-grafana \
-n monitoring \
-o jsonpath="{.data.admin-password}" | base64 --decode
```

---

# 🔐 Security Measures Implemented

* Static Code Analysis (SonarQube)
* Dependency Scan (OWASP)
* Container Scan (Trivy)
* No secrets in Docker images
* Kubernetes Secrets
* IAM OIDC integration
* HTTPS via ACM
* ALB Security Groups
* GitOps controlled deployments

---

# 📂 Repository Structure

```
wanderlust-app/
  ├── frontend/
  ├── backend/
  ├── Jenkinsfile
  ├── docker/
  └── README.md

wanderlust-gitops/
  ├── backend.yml
  ├── frontend.yml
  ├── ingress.yml
  ├── configmap.yml
  └── secret.yml
```

---

# 🔍 Verification Commands

```bash
kubectl get pods -n wanderlust
kubectl get svc -n wanderlust
kubectl get ingress -n wanderlust
kubectl get nodes
```

Check ALB:

```bash
kubectl describe ingress wanderlust-ingress -n wanderlust
```

Check ArgoCD:

```bash
argocd app list
argocd app get wanderlust
```

---

# 🧹 Cleanup & Cost Control

```bash
eksctl delete cluster --name wanderlust --region us-east-2
```

Also delete:

* ALB
* EBS volumes
* IAM roles (if manual)
* Unused security groups

---

# 🎯 Key Learning Outcomes

* Implemented full CI/CD automation
* Integrated DevSecOps tools
* Configured ALB with SSL
* Implemented GitOps deployment
* Used Kubernetes ConfigMap & Secrets
* Integrated Monitoring stack
* Understood IAM OIDC integration
* Production-level container orchestration

---

# 🔮 Future Improvements

* Implement Horizontal Pod Autoscaler
* Use Helm for application deployment
* Add Blue-Green deployment
* Implement WAF for ALB
* Use Terraform for infrastructure provisioning
* Add centralized logging (ELK)

---

# 📌 Final Summary

This project demonstrates a production-ready DevSecOps workflow where:

* MERN application is containerized
* Security scanning is integrated
* Deployment is automated using GitOps
* Infrastructure is managed on EKS
* Traffic is secured via ALB + SSL
* Monitoring is implemented via Prometheus & Grafana

This is a complete end-to-end cloud-native implementation.

---


