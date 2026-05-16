<![CDATA[<div align="center">

# ☸️ K8s Polling Platform

### A Cloud-Native Polling Application Deployed on Kubernetes (AWS EKS)

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](https://www.mongodb.com/)
[![Go](https://img.shields.io/badge/Go-00ADD8?style=for-the-badge&logo=go&logoColor=white)](https://golang.org/)
[![React](https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://reactjs.org/)
[![AWS EKS](https://img.shields.io/badge/AWS_EKS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/eks/)

> **Built & Maintained by [Saurabh Singh Rajput](https://github.com/N4si)**

---

*Vote for your favourite programming language through a beautiful web UI.  
Everything runs on Kubernetes — from the React frontend, through the Go API, down to a MongoDB replica set.*

</div>

---

## 📖 Table of Contents

| # | Section |
|---|---------|
| 1 | [What Does This Project Do?](#-what-does-this-project-do) |
| 2 | [Architecture Overview](#-architecture-overview) |
| 3 | [Tech Stack Explained](#-tech-stack-explained) |
| 4 | [Project Structure — Every File Explained](#-project-structure--every-file-explained) |
| 5 | [Kubernetes Concepts You'll Learn](#-kubernetes-concepts-youll-learn) |
| 6 | [Prerequisites](#-prerequisites) |
| 7 | [Step-by-Step Deployment Guide](#-step-by-step-deployment-guide) |
| 8 | [Verifying the Deployment](#-verifying-the-deployment) |
| 9 | [Troubleshooting](#-troubleshooting) |
| 10 | [Cleanup](#-cleanup) |
| 11 | [What You Will Learn](#-what-you-will-learn) |

---

## 🎯 What Does This Project Do?

This is a **polling web application** where users can vote for their favourite programming language out of six options: **C#, Python, JavaScript, Go, Java, and Node.js**.

**How it works in simple terms:**

1. A user opens the website in their browser.
2. The **React Frontend** displays six programming languages with a "+1" button next to each.
3. When a user clicks "+1", the frontend sends a request to the **Go API**.
4. The Go API receives the request and updates the vote count in the **MongoDB database**.
5. The updated vote count is sent back and displayed on the screen.

All three components (Frontend, API, Database) run as **containers inside a Kubernetes cluster** on AWS EKS.

---

## 🏗 Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        AWS EKS Cluster                         │
│                   Namespace: polling-platform                  │
│                                                                │
│  ┌──────────────┐     ┌──────────────┐     ┌────────────────┐  │
│  │   Frontend    │     │   Go API     │     │   MongoDB      │  │
│  │  (React App)  │────▶│  (REST API)  │────▶│ (Replica Set)  │  │
│  │  2 Replicas   │     │  2 Replicas  │     │  3 Replicas    │  │
│  └──────┬───────┘     └──────┬───────┘     └────────────────┘  │
│         │                    │                                  │
│  ┌──────▼───────┐     ┌──────▼───────┐     ┌────────────────┐  │
│  │ LoadBalancer  │     │ LoadBalancer  │     │Headless Service│  │
│  │  Service      │     │  Service      │     │ (clusterIP:    │  │
│  │  (port 80)    │     │  (port 80)    │     │      None)     │  │
│  └──────────────┘     └──────────────┘     └────────────────┘  │
│                                                                │
└─────────────────────────────────────────────────────────────────┘
         ▲                      ▲
         │                      │
    User browses           Frontend sends
    the website            AJAX vote requests
```

**Data Flow:**  
`User's Browser` → `Frontend ELB (port 80)` → `Frontend Pod (port 8080)` → `API ELB (port 80)` → `API Pod (port 8080)` → `MongoDB (port 27017)`

---

## 🛠 Tech Stack Explained

| Technology | Role | Why It's Used |
|-----------|------|---------------|
| **React + JavaScript** | Frontend UI | Builds a responsive, interactive single-page application for voting |
| **Go (Golang)** | Backend REST API | Fast, compiled language perfect for high-performance APIs |
| **MongoDB 4.2** | Database | NoSQL document database — stores votes as JSON-like documents |
| **Kubernetes (K8s)** | Container Orchestration | Manages, scales, and heals all the containers automatically |
| **AWS EKS** | Managed Kubernetes | Amazon's managed K8s service — handles the control plane for you |
| **Docker** | Containerisation | Packages each service into portable, reproducible containers |

---

## 📁 Project Structure — Every File Explained

```
k8s-polling-platform/
├── README.md                            ← You are here
└── manifests/                           ← All Kubernetes YAML files
    ├── 01-namespace.yaml                ← Creates isolated environment
    ├── 02-mongo-secret.yaml             ← Database credentials (base64)
    ├── 03-mongo-statefulset.yaml        ← MongoDB 3-node replica set
    ├── 04-mongo-service.yaml            ← Internal DNS for MongoDB pods
    ├── 05-api-deployment.yaml           ← Go REST API (2 replicas)
    ├── 06-api-service.yaml              ← Exposes API to the internet
    ├── 07-frontend-deployment.yaml      ← React app (2 replicas)
    └── 08-frontend-service.yaml         ← Exposes frontend to the internet
```

> **Why are files numbered 01–08?** They must be applied in this order. Each file depends on resources created by earlier files.

### Detailed File Breakdown

---

#### 📄 `01-namespace.yaml` — The Isolated Environment

**What is a Namespace?**  
Think of it as a **folder** inside your Kubernetes cluster. All resources for this project live inside the `polling-platform` namespace, keeping them separate from other projects.

**What this file does:**  
- Creates a namespace called `polling-platform`
- Adds labels so you can identify which project owns these resources

**Key fields:**
```yaml
kind: Namespace          # The type of resource to create
metadata:
  name: polling-platform # The name of our isolated environment
```

---

#### 📄 `02-mongo-secret.yaml` — Database Credentials

**What is a Secret?**  
A Kubernetes Secret stores sensitive data (passwords, API keys, tokens) separately from your application code. Pods can reference Secrets to get credentials without hardcoding them.

**What this file does:**  
- Stores the MongoDB username (`admin`) and password (`password`) in Base64 encoding
- The API deployment references this Secret to authenticate with MongoDB

**Key fields:**
```yaml
kind: Secret
data:
  username: YWRtaW4=      # base64 encoded "admin"
  password: cGFzc3dvcmQ=  # base64 encoded "password"
```

> ⚠️ **Important:** Base64 is **encoding**, NOT **encryption**. Anyone can decode it. In production, use tools like HashiCorp Vault or AWS Secrets Manager.

---

#### 📄 `03-mongo-statefulset.yaml` — MongoDB Replica Set (3 Pods)

**What is a StatefulSet?**  
Unlike a Deployment (which treats all Pods as interchangeable), a StatefulSet gives each Pod a **stable identity** — `mongo-0`, `mongo-1`, `mongo-2`. This is essential for databases because each replica needs a unique, predictable hostname.

**What is a Replica Set (MongoDB)?**  
MongoDB's built-in replication feature. One node is the **Primary** (handles writes), and two are **Secondaries** (replicate data from Primary). If the Primary crashes, a Secondary is automatically elected as the new Primary.

**What this file does:**  
- Creates 3 MongoDB Pods with stable hostnames
- Each Pod gets its own 500 MB persistent disk (data survives Pod restarts)
- Uses `podAntiAffinity` to spread Pods across different Nodes
- Configures MongoDB with `--replSet rs0` to enable replication

**Key fields:**
```yaml
kind: StatefulSet
spec:
  serviceName: mongo      # Links to the Headless Service
  replicas: 3             # 1 Primary + 2 Secondary
  volumeClaimTemplates:   # Each Pod gets its own persistent disk
    - spec:
        storage: 0.5Gi    # 500 MB per Pod
```

---

#### 📄 `04-mongo-service.yaml` — Headless Service for MongoDB

**What is a Headless Service?**  
A normal Service load-balances traffic randomly across Pods. A **Headless** Service (`clusterIP: None`) creates individual DNS records for each Pod instead:

- `mongo-0.mongo` → IP of mongo-0
- `mongo-1.mongo` → IP of mongo-1  
- `mongo-2.mongo` → IP of mongo-2

**Why does MongoDB need this?**  
MongoDB clients must know the address of **every** replica set member so they can send writes to the Primary and reads to Secondaries. Random load balancing would break replication.

**Key fields:**
```yaml
kind: Service
spec:
  clusterIP: None     # ← This makes it Headless
  selector:
    role: db           # Routes to MongoDB Pods
```

---

#### 📄 `05-api-deployment.yaml` — Go REST API

**What is a Deployment?**  
A Deployment tells Kubernetes: "I want X copies of this container running at all times." If a Pod crashes, Kubernetes creates a new one automatically.

**What this file does:**  
- Runs 2 replicas of the Go API for high availability
- Connects to MongoDB using the connection string and Secret credentials
- Uses `RollingUpdate` strategy — new versions are deployed without downtime
- **Liveness Probe:** Kubernetes restarts the Pod if `/ok` stops responding
- **Readiness Probe:** Kubernetes stops sending traffic until the Pod is healthy

**API Endpoints:**
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/ok` | Health check |
| GET | `/languages` | List all languages with vote counts |
| GET | `/languages/{name}` | Get details for one language |
| POST | `/languages/{name}` | Increment vote count |

---

#### 📄 `06-api-service.yaml` — Expose API to the Internet

**What is a LoadBalancer Service?**  
It tells your cloud provider (AWS) to create a real, internet-facing load balancer (ELB). External traffic comes in on port 80 and gets forwarded to port 8080 inside the Pods.

**Traffic flow:**
```
Internet → AWS ELB (port 80) → K8s Service → API Pod (port 8080)
```

---

#### 📄 `07-frontend-deployment.yaml` — React Frontend

**What this file does:**  
- Runs 2 replicas of the React app
- The `REACT_APP_APIHOSTPORT` environment variable tells the frontend where the API is
- Same health probes as the API deployment

> ⚠️ You must set `REACT_APP_APIHOSTPORT` to the API's Load Balancer DNS name after deploying the API service.

---

#### 📄 `08-frontend-service.yaml` — Expose Frontend to the Internet

**What this file does:**  
- Creates another LoadBalancer that users access from their browser
- Same concept as the API Service, but for the React frontend

**Traffic flow:**
```
User's Browser → AWS ELB (port 80) → K8s Service → Frontend Pod (port 8080)
```

---

## 🧠 Kubernetes Concepts You'll Learn

| Concept | What It Is | Where It's Used |
|---------|-----------|-----------------|
| **Namespace** | Isolated virtual cluster | `01-namespace.yaml` |
| **Secret** | Stores sensitive data (passwords/keys) | `02-mongo-secret.yaml` |
| **StatefulSet** | Deployment with stable Pod identities | `03-mongo-statefulset.yaml` |
| **Headless Service** | DNS-per-Pod (no load balancing) | `04-mongo-service.yaml` |
| **Deployment** | Stateless app with replicas & auto-healing | `05-api-deployment.yaml`, `07-frontend-deployment.yaml` |
| **LoadBalancer Service** | Exposes Pods to the internet via cloud LB | `06-api-service.yaml`, `08-frontend-service.yaml` |
| **PersistentVolumeClaim** | Requests persistent disk storage | Inside `03-mongo-statefulset.yaml` |
| **RollingUpdate** | Zero-downtime deployment strategy | Inside Deployments |
| **Liveness Probe** | Auto-restart unhealthy Pods | Inside Deployments |
| **Readiness Probe** | Only send traffic to ready Pods | Inside Deployments |
| **Pod Anti-Affinity** | Spread Pods across Nodes for HA | Inside `03-mongo-statefulset.yaml` |

---

## ✅ Prerequisites

Before you begin, make sure you have:

| Tool | Purpose | Install Guide |
|------|---------|---------------|
| **AWS Account** | Cloud infrastructure | [aws.amazon.com](https://aws.amazon.com/) |
| **AWS CLI v2** | Interact with AWS from terminal | [Install Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) |
| **kubectl** | Interact with Kubernetes cluster | [Install Guide](https://kubernetes.io/docs/tasks/tools/) |
| **AWS EKS Cluster** | Managed Kubernetes cluster | Created in Step 1 below |

---

## 🚀 Step-by-Step Deployment Guide

### Step 1 — Create the EKS Cluster

Create an EKS cluster with a Node Group of **2 × t2.medium** instances:

```bash
# Using AWS Console or eksctl:
eksctl create cluster \
  --name polling-platform-cluster \
  --region us-west-2 \
  --nodegroup-name workers \
  --node-type t2.medium \
  --nodes 2
```

### Step 2 — Install CLI Tools (if not already installed)

**Install kubectl:**
```bash
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.11/2023-03-17/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo cp ./kubectl /usr/local/bin
export PATH=/usr/local/bin:$PATH
```

**Install AWS CLI:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Step 3 — Connect to the Cluster

```bash
aws eks update-kubeconfig --name polling-platform-cluster --region us-west-2
```

**Verify connection:**
```bash
kubectl get nodes
# You should see 2 nodes in "Ready" status
```

> If you get `Unauthorized`, see [AWS troubleshooting guide](https://repost.aws/knowledge-center/eks-api-server-unauthorized-error).

### Step 4 — Clone This Repository

```bash
git clone https://github.com/N4si/k8s-polling-platform.git
cd k8s-polling-platform/manifests
```

### Step 5 — Create the Namespace

```bash
kubectl apply -f 01-namespace.yaml

# Switch to the new namespace
kubectl config set-context --current --namespace polling-platform
```

### Step 6 — Deploy MongoDB

**6a. Create the Secret:**
```bash
kubectl apply -f 02-mongo-secret.yaml
```

**6b. Create the StatefulSet (3 MongoDB Pods):**
```bash
kubectl apply -f 03-mongo-statefulset.yaml
```

**6c. Create the Headless Service:**
```bash
kubectl apply -f 04-mongo-service.yaml
```

**6d. Verify DNS resolution:**
```bash
# Start a temporary pod to test DNS
kubectl run --rm utils -it --image praqma/network-multitool -- bash

# Inside the pod, run:
for i in {0..2}; do nslookup mongo-$i.mongo; done

# You should see IP addresses for all 3 mongo pods
# Type 'exit' to leave
```

**6e. Initialise the MongoDB Replica Set:**
```bash
cat <<EOF | kubectl exec -it mongo-0 -- mongo
rs.initiate();
sleep(2000);
rs.add("mongo-1.mongo:27017");
sleep(2000);
rs.add("mongo-2.mongo:27017");
sleep(2000);
cfg = rs.conf();
cfg.members[0].host = "mongo-0.mongo:27017";
rs.reconfig(cfg, {force: true});
sleep(5000);
EOF
```
> ⏳ Wait ~15 seconds. It completes when you see `bye`.

**6f. Verify the replica set:**
```bash
kubectl exec -it mongo-0 -- mongo --eval "rs.status()" | grep "PRIMARY\|SECONDARY"
# Expected: 1 PRIMARY, 2 SECONDARY
```

**6g. Load sample data into MongoDB:**
```bash
cat <<EOF | kubectl exec -it mongo-0 -- mongo
use langdb;
db.languages.insert({"name" : "csharp", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 5, "compiled" : false, "homepage" : "https://dotnet.microsoft.com/learn/csharp", "download" : "https://dotnet.microsoft.com/download/", "votes" : 0}});
db.languages.insert({"name" : "python", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 3, "script" : false, "homepage" : "https://www.python.org/", "download" : "https://www.python.org/downloads/", "votes" : 0}});
db.languages.insert({"name" : "javascript", "codedetail" : { "usecase" : "web, client-side", "rank" : 7, "script" : false, "homepage" : "https://en.wikipedia.org/wiki/JavaScript", "download" : "n/a", "votes" : 0}});
db.languages.insert({"name" : "go", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 12, "compiled" : true, "homepage" : "https://golang.org", "download" : "https://golang.org/dl/", "votes" : 0}});
db.languages.insert({"name" : "java", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 1, "compiled" : true, "homepage" : "https://www.java.com/en/", "download" : "https://www.java.com/en/download/", "votes" : 0}});
db.languages.insert({"name" : "nodejs", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 20, "script" : false, "homepage" : "https://nodejs.org/en/", "download" : "https://nodejs.org/en/download/", "votes" : 0}});
db.languages.find().pretty();
EOF
```

### Step 7 — Deploy the Go API

```bash
kubectl apply -f 05-api-deployment.yaml
kubectl apply -f 06-api-service.yaml
```

**Wait for the Load Balancer and test:**
```bash
API_ELB_PUBLIC_FQDN=$(kubectl get svc api -ojsonpath="{.status.loadBalancer.ingress[0].hostname}")
until nslookup $API_ELB_PUBLIC_FQDN >/dev/null 2>&1; do sleep 2 && echo "waiting for DNS..."; done
echo "API is ready at: http://$API_ELB_PUBLIC_FQDN"
```

**Test the API endpoints:**
```bash
curl -s $API_ELB_PUBLIC_FQDN/ok
curl -s $API_ELB_PUBLIC_FQDN/languages | jq .
curl -s $API_ELB_PUBLIC_FQDN/languages/go | jq .
```

### Step 8 — Deploy the React Frontend

```bash
kubectl apply -f 07-frontend-deployment.yaml
kubectl apply -f 08-frontend-service.yaml
```

**Get the Frontend URL:**
```bash
FRONTEND_ELB_PUBLIC_FQDN=$(kubectl get svc frontend -ojsonpath="{.status.loadBalancer.ingress[0].hostname}")
until nslookup $FRONTEND_ELB_PUBLIC_FQDN >/dev/null 2>&1; do sleep 2 && echo "waiting for DNS..."; done
echo "Open in your browser: http://$FRONTEND_ELB_PUBLIC_FQDN"
```

🎉 **Open that URL in your browser and start voting!**

---

## ✔️ Verifying the Deployment

**Check all Pods are running:**
```bash
kubectl get pods -n polling-platform
# Expected: mongo-0, mongo-1, mongo-2, api-xxx, api-xxx, frontend-xxx, frontend-xxx
```

**Check all Services:**
```bash
kubectl get svc -n polling-platform
# Expected: mongo (ClusterIP None), api (LoadBalancer), frontend (LoadBalancer)
```

**Verify votes are being saved in MongoDB:**
```bash
kubectl exec -it mongo-0 -- mongo langdb --eval "db.languages.find().pretty()"
```

---

## 🔧 Troubleshooting

| Problem | Solution |
|---------|----------|
| `Unauthorized` error with kubectl | [Follow this AWS guide](https://repost.aws/knowledge-center/eks-api-server-unauthorized-error) |
| Pods stuck in `Pending` | Run `kubectl describe pod <pod-name>` — likely insufficient Node resources |
| LoadBalancer has no external IP | Wait 2–3 minutes for AWS to provision the ELB |
| Frontend shows no data | Check `REACT_APP_APIHOSTPORT` is set to the API's ELB DNS |
| MongoDB Pods in `CrashLoopBackOff` | Check logs: `kubectl logs mongo-0` — might be a storage class issue |

---

## 🧹 Cleanup

To delete all resources and avoid AWS charges:

```bash
# Delete all resources in the namespace
kubectl delete namespace polling-platform

# Delete the EKS cluster (if you created it with eksctl)
eksctl delete cluster --name polling-platform-cluster --region us-west-2
```

---

## 🎓 What You Will Learn

By completing this project, you will gain hands-on experience with:

1. **Containerisation** — How apps are packaged into Docker containers
2. **Kubernetes Orchestration** — Managing containers at scale with K8s
3. **Microservices Architecture** — Frontend, API, and DB as independent services
4. **Database Replication** — Setting up a MongoDB replica set for high availability
5. **Secrets Management** — Storing credentials securely in Kubernetes
6. **Stateful vs Stateless** — When to use StatefulSet vs Deployment
7. **Persistent Storage** — How Kubernetes manages disk volumes
8. **Health Probes** — Liveness and Readiness checks for self-healing
9. **Service Networking** — LoadBalancer vs Headless Services
10. **Rolling Updates** — Zero-downtime deployment strategies

---

<div align="center">

**⭐ Star this repo if it helped you learn Kubernetes! ⭐**

Built with ❤️ by **[Saurabh Singh Rajput](https://github.com/N4si)**

</div>
]]>
