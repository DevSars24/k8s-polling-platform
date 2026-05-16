# ☸️ K8s Polling Platform

<div align="center">

### Cloud-Native Polling Application on Kubernetes (AWS EKS)

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge\&logo=kubernetes\&logoColor=white)](https://kubernetes.io/)
[![AWS EKS](https://img.shields.io/badge/AWS_EKS-FF9900?style=for-the-badge\&logo=amazonaws\&logoColor=white)](https://aws.amazon.com/eks/)
[![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge\&logo=mongodb\&logoColor=white)](https://mongodb.com/)
[![React](https://img.shields.io/badge/React-61DAFB?style=for-the-badge\&logo=react\&logoColor=black)](https://react.dev/)
[![Go](https://img.shields.io/badge/Go-00ADD8?style=for-the-badge\&logo=go\&logoColor=white)](https://go.dev/)

### 🚀 Full Stack Microservices Application Deployed on Kubernetes

Vote for your favourite programming language through a modern React frontend powered by a Go backend API and MongoDB replica set running on Kubernetes.

**Built by [Saurabh Singh](https://github.com/DevSars24)**

</div>

---

# 📚 Table of Contents

* [Overview](#-overview)
* [Architecture](#-architecture)
* [Tech Stack](#-tech-stack)
* [Project Structure](#-project-structure)
* [Kubernetes Concepts Used](#-kubernetes-concepts-used)
* [Prerequisites](#-prerequisites)
* [Deployment Guide](#-deployment-guide)
* [Verification](#-verification)
* [Troubleshooting](#-troubleshooting)
* [Cleanup](#-cleanup)
* [What You’ll Learn](#-what-youll-learn)

---

# 🎯 Overview

This project is a **cloud-native polling platform** built using:

* **React** frontend
* **Go REST API**
* **MongoDB Replica Set**
* **Kubernetes on AWS EKS**

Users can vote for different programming languages through a responsive web interface.

The complete application is containerized and orchestrated using Kubernetes.

---

# ⚙️ How It Works

1. User opens the frontend in the browser
2. React frontend sends API requests
3. Go backend processes the requests
4. MongoDB stores vote data
5. Updated results are returned to the frontend

---

# 🏗 Architecture

```text
┌──────────────────────────────────────────────┐
│                AWS EKS Cluster               │
│                                              │
│   ┌─────────────┐      ┌─────────────┐       │
│   │  Frontend   │────▶ │   Go API    │────▶ │
│   │   React     │      │  Backend    │      │
│   └─────────────┘      └─────────────┘      │
│                                 │            │
│                                 ▼            │
│                        ┌────────────────┐    │
│                        │   MongoDB RS   │    │
│                        │  StatefulSet   │    │
│                        └────────────────┘    │
│                                              │
└──────────────────────────────────────────────┘
```

---

# 🛠 Tech Stack

| Technology  | Purpose                    |
| ----------- | -------------------------- |
| React       | Frontend UI                |
| Go (Golang) | Backend API                |
| MongoDB     | Database                   |
| Docker      | Containerization           |
| Kubernetes  | Container Orchestration    |
| AWS EKS     | Managed Kubernetes Cluster |

---

# 📁 Project Structure

```text
k8s-polling-platform/
│
├── README.md
│
└── manifests/
    ├── 01-namespace.yaml
    ├── 02-mongo-secret.yaml
    ├── 03-mongo-statefulset.yaml
    ├── 04-mongo-service.yaml
    ├── 05-api-deployment.yaml
    ├── 06-api-service.yaml
    ├── 07-frontend-deployment.yaml
    └── 08-frontend-service.yaml
```

---

# ☸️ Kubernetes Concepts Used

| Concept            | Description                          |
| ------------------ | ------------------------------------ |
| Namespace          | Isolated environment                 |
| Secret             | Secure credential storage            |
| StatefulSet        | MongoDB replica management           |
| Deployment         | Stateless application deployment     |
| Service            | Internal & external networking       |
| LoadBalancer       | Public application access            |
| Persistent Volumes | Persistent MongoDB storage           |
| Liveness Probe     | Automatic pod health checks          |
| Readiness Probe    | Traffic routing only to healthy pods |

---

# ✅ Prerequisites

Install the following tools before starting:

* AWS CLI
* kubectl
* eksctl
* Docker
* AWS Account

---

# 🚀 Deployment Guide

## 1️⃣ Create EKS Cluster

```bash
eksctl create cluster \
  --name polling-platform-cluster \
  --region us-west-2 \
  --nodegroup-name workers \
  --node-type t2.medium \
  --nodes 2
```

---

## 2️⃣ Configure kubectl

```bash
aws eks update-kubeconfig \
  --name polling-platform-cluster \
  --region us-west-2
```

Verify cluster connection:

```bash
kubectl get nodes
```

---

## 3️⃣ Clone Repository

```bash
git clone https://github.com/DevSars24/k8s-polling-platform.git

cd k8s-polling-platform/manifests
```

---

## 4️⃣ Create Namespace

```bash
kubectl apply -f 01-namespace.yaml

kubectl config set-context --current --namespace polling-platform
```

---

## 5️⃣ Deploy MongoDB

```bash
kubectl apply -f 02-mongo-secret.yaml

kubectl apply -f 03-mongo-statefulset.yaml

kubectl apply -f 04-mongo-service.yaml
```

Check pods:

```bash
kubectl get pods
```

---

## 6️⃣ Initialize MongoDB Replica Set

```bash
kubectl exec -it mongo-0 -- mongo
```

Inside Mongo shell:

```javascript
rs.initiate()

rs.add("mongo-1.mongo:27017")

rs.add("mongo-2.mongo:27017")
```

---

## 7️⃣ Deploy Backend API

```bash
kubectl apply -f 05-api-deployment.yaml

kubectl apply -f 06-api-service.yaml
```

Get API LoadBalancer:

```bash
kubectl get svc
```

---

## 8️⃣ Deploy Frontend

```bash
kubectl apply -f 07-frontend-deployment.yaml

kubectl apply -f 08-frontend-service.yaml
```

---

# 🌐 Access Application

Get frontend LoadBalancer URL:

```bash
kubectl get svc frontend
```

Open the external IP/DNS in your browser.

---

# ✔️ Verification

Check all resources:

```bash
kubectl get all
```

Check services:

```bash
kubectl get svc
```

Check pods:

```bash
kubectl get pods
```

---

# 🔧 Troubleshooting

| Issue                      | Solution               |
| -------------------------- | ---------------------- |
| Pods stuck in Pending      | Check node resources   |
| CrashLoopBackOff           | Check pod logs         |
| LoadBalancer not available | Wait a few minutes     |
| Frontend not loading data  | Verify API endpoint    |
| MongoDB issues             | Check StatefulSet logs |

View logs:

```bash
kubectl logs <pod-name>
```

---

# 🧹 Cleanup

Delete namespace:

```bash
kubectl delete namespace polling-platform
```

Delete EKS cluster:

```bash
eksctl delete cluster \
  --name polling-platform-cluster \
  --region us-west-2
```

---

# 🎓 What You’ll Learn

By building this project, you’ll learn:

* Kubernetes fundamentals
* Deployments & StatefulSets
* Services & networking
* MongoDB replica sets
* Cloud-native architecture
* AWS EKS deployment
* Container orchestration
* Scaling microservices
* Persistent storage handling
* Production-style DevOps workflows

---

<div align="center">

## ⭐ If this project helped you, consider starring the repository!

Built with ❤️ by **Saurabh Singh**

</div>

