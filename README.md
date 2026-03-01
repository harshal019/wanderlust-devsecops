# 🌍 Wanderlust – Production Grade MERN DevSecOps Deployment on AWS EKS

🔗 **Live Application:**  
https://wanderlust.harshalgharat.site  

![Preview Image](https://github.com/krishnaacharyaa/wanderlust/assets/116620586/17ba9da6-225f-481d-87c0-5d5a010a9538)


Deployed on Amazon EKS using ALB Ingress and secured with AWS Certificate Manager (HTTPS).

---

# 📚 Project Navigation

| Section | Description |
|----------|------------|
| 🚀 Live Application | Production URL with SSL |
| 📌 Project Overview | High-level system explanation |
| 🏗 Architecture | Complete system flow |
| 🌐 Traffic Flow | Request routing inside cluster |
| 🧱 Tech Stack | Technologies used |
| 🔐 CI Pipeline | DevSecOps implementation |
| 🔄 CD + GitOps | Deployment automation |
| ☁ Infrastructure Setup | EKS, OIDC & ALB setup |
| 📊 Monitoring | Prometheus & Grafana |
| 🔑 Configuration | ConfigMap & Secrets |
| 🏛 Backend Design | MVC architecture |
| 🔥 Challenges Solved | Real-world debugging |
| 📸 Screenshots | Proof of deployment |
| 🚀 Future Improvements | Scalability roadmap |

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

## 1️⃣ Create EKS Cluster

```bash
eksctl create cluster \
  --name wanderlust \
  --region us-east-2 \
  --without-nodegroup

---

### 2️⃣ Associate OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster wanderlust \
  --region us-east-2 \
  --approve

---

### 3️⃣ Create Nodegroup

```bash
eksctl create nodegroup \
  --cluster wanderlust \
  --name wanderlust-ng \
  --node-type t2.large \
  --nodes 2

---

### 4️⃣ Install AWS Load Balancer Controller (Helm)

helm repo add eks https://aws.github.io/eks-charts

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=wanderlust

ALB is automatically created when Ingress resource is applied.

---

# 🔄 GitOps Deployment (ArgoCD)

## Install ArgoCD

kubectl create namespace argocd

kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

---

## Import EKS Cluster

argocd cluster add \
  Wanderlust@wanderlust.us-east-2.eksctl.io \
  --name wanderlust-eks

ArgoCD continuously monitors GitOps repository and syncs application changes automatically.

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

---

## Jenkins CD Stages

- Update Kubernetes Image Tag
- Push to GitOps Repository
- Trigger ArgoCD Sync
- Rolling Update Deployment

---

# 📊 Monitoring (Helm Based)

helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts

kubectl create namespace prometheus

helm install monitoring \
  prometheus-community/kube-prometheus-stack \
  -n prometheus

Prometheus and Grafana exposed via ALB Ingress.

---

# 🔑 Configuration Management

- Docker images contain no environment variables
- ConfigMap used for application config
- Kubernetes Secret used for sensitive values (JWT, credentials)
- SSL handled via ACM
- Path-based routing via Ingress

---

# 🏛 Backend Architecture (MVC Pattern)

Client  
↓  
Routes  
↓  
Controllers  
↓  
Services  
↓  
Models  
↓  
MongoDB  

Redis middleware used for caching frequently accessed endpoints.

---

# 🔥 Challenges Solved

- Fixed ALB not creating due to missing OIDC association
- Resolved Ingress path routing conflict (/api duplication issue)
- Debugged Service port mismatch (80 vs 5173)
- Fixed Target Group unhealthy issue
- Implemented secure SSL termination via ACM
- Configured GitOps auto-sync workflow

---

# 🚀 Future Improvements

- Horizontal Pod Autoscaler (HPA)
- Use RDS instead of in-cluster MongoDB
- Private subnets for backend services
- Add CI caching optimization
- Implement Canary deployment strategy

---

# 👨‍💻 Author

Harshal Gharat  
DevOps | Kubernetes | AWS | DevSecOps