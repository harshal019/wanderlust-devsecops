# 🌍 Wanderlust – Production Grade MERN DevSecOps Deployment on AWS EKS

![EKS](https://img.shields.io/badge/AWS-EKS-orange?logo=amazon-aws)
![Jenkins](https://img.shields.io/badge/CI/CD-Jenkins-red?logo=jenkins)
![ArgoCD](https://img.shields.io/badge/GitOps-ArgoCD-blue?logo=argo)
![Kubernetes](https://img.shields.io/badge/Orchestration-Kubernetes-blue?logo=kubernetes)
![Docker](https://img.shields.io/badge/Container-Docker-blue?logo=docker)

🔗 **Live Application:**  
https://wanderlust.harshalgharat.site  

![Home Page](Assets/screenshots/01-app-live.png)

![Post / Blog Page](Assets/screenshots/01-app-live-2.png)

Deployed on Amazon EKS using ALB Ingress and secured with AWS Certificate Manager (HTTPS).

> A full-stack travel blog deployed end-to-end on AWS EKS with automated 
> CI/CD, security scanning, GitOps sync, and real-time monitoring — 
> zero manual deployments.
---

## 📚 Contents

- [Overview](#-project-overview)
- [Architecture](#-high-level-architecture)
- [Tech Stack](#-tech-stack)
- [Infrastructure Setup](#-infrastructure-setup)
- [AWS Load Balancer Controller](#-aws-load-balancer-controller-setup)
- [GitOps with ArgoCD](#-gitops-with-argocd)
- [CI/CD Pipeline](#-cicd-pipeline-devsecops)
- [Monitoring Setup](#-monitoring-prometheus--grafana)
- [Configuration Management](#-configuration-management)
- [Challenges Solved](#-challenges-solved)
- [Future Improvements](#-future-improvements)


---

# 📌 Project Overview

Wanderlust is a production-ready MERN travel blog application deployed on AWS using a complete **DevSecOps + GitOps pipeline**.

This project demonstrates:

- Containerized MERN application
- Automated CI/CD using Jenkins
- DevSecOps security scanning
- Kubernetes deployment on Amazon EKS
- ALB-based Ingress with SSL
- GitOps deployment using ArgoCD
- Monitoring with Prometheus & Grafana
- Secure configuration using ConfigMaps & Secrets

---

# 🏗 High-Level Architecture

![Architecture](Assets/DevSecOps+GitOps.gif)

---

# 🧱 Tech Stack

## Application Stack

| Layer | Technology |
|--------|------------|
| Frontend | React (Vite) |
| Backend | Node.js + Express (MVC) |
| Database | MongoDB |
| Caching | Redis |
| Web Server | Nginx |

---

## DevOps & Cloud Stack

| Category | Tools Used |
|------------|------------|
| Containerization | Docker |
| CI Pipeline | Jenkins |
| Code Quality | SonarQube |
| Dependency Scan | OWASP Dependency Check |
| Image & FS Scan | Trivy |
| Container Registry | DockerHub |
| Orchestration | Amazon EKS |
| Ingress | AWS Load Balancer Controller |
| GitOps | ArgoCD |
| Monitoring | Prometheus + Grafana (Helm) |
| DNS | Route53 |
| SSL | AWS Certificate Manager |
| Configuration | ConfigMap + Kubernetes Secrets |

---

# ☁️ Infrastructure Setup

### 1️⃣ Create EKS Cluster

```bash
eksctl create cluster \
  --name wanderlust \
  --region us-east-2 \
  --without-nodegroup
```

---

### 2️⃣ Associate OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster wanderlust \
  --region us-east-2 \
  --approve
```

---

### 3️⃣ Create Nodegroup

```bash
eksctl create nodegroup --cluster=wanderlust \
                     --region=us-east-2 \
                     --name=wanderlust-ng \
                     --node-type=t2.medium \
                     --nodes=2 \
                     --nodes-min=2 \
                     --nodes-max=5 \
                     --node-volume-size=19 \
                     --ssh-access \
                     --ssh-public-key=blog-key 
```

---

### 4️⃣ Install AWS Load Balancer Controller (Helm)

```bash
helm repo add eks https://aws.github.io/eks-charts

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=wanderlust
```

ALB is automatically created when Ingress resource is applied.
---
### 🖼️ Ingress Resource
---
![Ingress ALB](Assets/screenshots/ingress.png)
---
### 🖼️ AWS ALB Console

![AWS ALB Console](Assets/screenshots/10-aws-alb-console.png)
---
### 🖼️ Route53 DNS Record

![Route53 Record](Assets/screenshots/route53.png)


---


# 🔄 GitOps Deployment (ArgoCD)

## Install ArgoCD

```bash
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
---

## Check Available Clusters

```bash
kubectl config get-contexts


```

---
## Import EKS Cluster into ArgoCD

```bash
argocd cluster add harshal-eks@wanderlust.us-east-2.eksctl.io --name wanderlust-eks
```

ArgoCD continuously syncs Kubernetes manifests from GitOps repository.

### 🖼️ ArgoCD — Synced & Healthy

![ArgoCD Sync](Assets/screenshots/06-argocd-sync.png)

---

# 🔐 CI/CD Pipeline (DevSecOps)

## Jenkins CI Stages

- Code Checkout
- SonarQube Analysis
- Quality Gate Validation
- OWASP Dependency Scan
- Trivy Filesystem Scan
- Docker Image Build
- Trivy Image Scan
- Push Versioned Image


### 🖼️ CI Pipeline — All Stages

![CI Pipeline](Assets/screenshots/02-jenkins-ci.png)

---
### 🖼️ SonarQube — Quality Gate Passed

![SonarQube](Assets/screenshots/03-sonarqube.png)

---

### 🖼️ DockerHub — Versioned Images

![DockerHub Images ](Assets/screenshots/dockerhub1.png)

---

![DockerHub Images](Assets/screenshots/dockerhub2.png)

---


## Jenkins CD Stages

- Update Kubernetes Image Tag
- Push to GitOps Repository
- Trigger ArgoCD Sync
- Rolling Update Deployment

### 🖼️ CD Pipeline — Deploy & Sync

![CD Pipeline](Assets/screenshots/02-jenkins-cd.png)

---


## ☸️ Kubernetes Cluster
> `kubectl get pods,svc -n wanderlust -o wide`
![K8s Pods & Services](Assets/screenshots/kubernetes.png)

---

# 📊 Monitoring (Helm Based)

```bash
helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts

kubectl create namespace prometheus

helm install monitoring \
  prometheus-community/kube-prometheus-stack \
  -n prometheus

```
Prometheus and Grafana exposed via ALB Ingress.

### 🖼️ Grafana — Cluster Dashboard

**Node Metrics Dashboard**
![Grafana — Node Metrics](Assets/screenshots/grafana-node-metrics.png)

---

**Kubernetes Cluster Dashboard**
![Grafana — Kubernetes Cluster](Assets/screenshots/grafana-kubernetes-cluster.png)
---
### 🖼️ Prometheus — All Targets UP

![Prometheus Targets](Assets/screenshots/prometheus.png)

---

# 🔑 Configuration Management

- No environment variables baked into Docker images
- ConfigMap for application configuration
- Kubernetes Secret for sensitive values
- SSL handled via ACM
- Path-based routing via Ingress

## 🖼️ ConfigMap & Secrets

![ConfigMap and Secret](Assets/screenshots/configmap-secret.png)
---


# 🔥 Challenges Solved

- Fixed ALB not creating due to missing OIDC association  
  → Added `eksctl utils associate-iam-oidc-provider` before controller install
- Resolved Ingress path routing conflict (/api duplication issue)  
  → Removed duplicate `/api` prefix from backend Ingress path rule
- Debugged Service port mismatch (80 vs 5173)  
  → Updated Service targetPort to match Vite dev server port 5173
- Fixed Target Group unhealthy issue  
  → Added correct health check path `/api/health` in Ingress annotations
- Implemented secure SSL termination via ACM  
  → Attached ACM certificate ARN via `alb.ingress.kubernetes.io/certificate-arn`
- Configured GitOps auto-sync workflow  
  → Set ArgoCD sync policy to automated with self-heal enabled

---

# 🚀 Future Improvements

- Horizontal Pod Autoscaler (HPA)
- Use RDS instead of in-cluster MongoDB
- Private subnets for backend services
- Add CI caching optimization
- Terraform-based infrastructure provisioning
---


# 👨‍💻 Author

**Harshal Gharat**  
DevOps | Kubernetes | AWS | DevSecOps  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin)](https://www.linkedin.com/in/harshalgharat01/)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black?logo=github)](https://github.com/harshal019)
