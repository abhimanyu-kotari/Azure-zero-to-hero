# 03 - Containers & Orchestration: Docker & Containerization Basics

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `03:47:55` to `04:23:32` (Docker, Containerization & AKS Orchestration)

---

## 1. The Real-World Analogy: Why Were Containers Invented?

In the video, M Prashant introduces containers with the classic historical problem of global freight shipping:

> *"Before the 1950s, cargo ships were loaded with loose boxes, barrels, sacks, and wooden crates of completely different shapes and sizes. Loading a ship took weeks, items fell and broke, and thieves stole cargo at the docks.*  
> *Then came the **Standard Steel Shipping Container**. Whether you are shipping cars, electronics, wheat, or clothes, everything is packed inside an identical standard steel box. The cargo ship, train, and truck do not care what is inside; they only know how to pick up and move the standard container!*  
> *In software, **Docker Containers** do the exact same thing. Instead of fighting with 'It works on my machine but crashes on the server!', you package your code, Python/Node runtime, system libraries, and dependencies into a standard container that runs identically on a MacBook, Windows laptop, or Microsoft Azure cloud!"*

```
+---------------------------------------------------------------------------------------------------+
|                                 THE SHIPPING CONTAINER ANALOGY                                    |
+---------------------------------------------------------------------------------------------------+
|  TRADITIONAL SOFTWARE SHIPMENT (Chaos)            DOCKER CONTAINER SHIPMENT (Standardized)        |
|                                                                                                   |
|  [ Python 3.9 ]  [ Node 18 ]  [ Debian Libs ]     +--------------------------------------------+  |
|  \             |             /                    |             DOCKER CONTAINER               |  |
|   \            |            /                     |  - Application Source Code                 |  |
|    v           v           v                      |  - Exact Runtime & Dependencies            |  |
|  "Crashes due to missing system library           |  - Configuration & Environment Variables   |  |
|   or OS version mismatch on production!"          +--------------------------------------------+  |
|                                                                 |                                 |
|                                                                 v                                 |
|                                                   Runs 100% identically on ANY host!              |
+---------------------------------------------------------------------------------------------------+
```

---

## 2. Virtual Machines vs. Docker Containers

To ace cloud interviews and build modern applications, every engineer must understand the core architectural difference between Virtual Machines and Containers:

```
+-----------------------------------+     +-----------------------------------+
|         VIRTUAL MACHINES          |     |            CONTAINERS             |
+-----------------------------------+     +-----------------------------------+
|  [App A]      [App B]     [App C] |     |  [App A]      [App B]     [App C] |
|  [Bins/Libs]  [Bins/Libs] [Bins]  |     |  [Bins/Libs]  [Bins/Libs] [Bins]  |
|  [Guest OS]   [Guest OS]  [Guest] |     +-----------------------------------+
|  (Ubuntu 4GB) (Debian 2GB)(Win8GB)|     |           DOCKER ENGINE           |
+-----------------------------------+     +-----------------------------------+
|            HYPERVISOR             |     |      HOST OPERATING SYSTEM        |
+-----------------------------------+     +-----------------------------------+
|         PHYSICAL HARDWARE         |     |         PHYSICAL HARDWARE         |
+-----------------------------------+     +-----------------------------------+
```

| Architectural Metric | Virtual Machine (IaaS) | Docker Container |
| :--- | :--- | :--- |
| **Virtualization Layer** | Hardware-level (Virtualizes CPU, RAM, Hard Disk, NIC) | **OS-level** (Shares the host Linux kernel) |
| **Operating System** | Requires a dedicated **Guest OS** for every single VM | **No Guest OS**; uses host kernel namespaces & cgroups |
| **Boot-up Speed** | **1 to 5 minutes** (Full OS boot sequence) | **Milliseconds to seconds** (Instant process start) |
| **Disk Size** | **10 GB to 100 GB** per VM | **20 MB to 500 MB** per container image |
| **Memory Consumption** | High (Dozens of background OS services consume RAM) | Low (Only consumes memory needed by your app code) |
| **Density & Scaling** | 5 to 10 VMs per physical host | **100+ containers** per physical host |
| **Portability** | Hypervisor-specific (VHD, VMDK) | **Universal standard** (OCI compliant, runs anywhere) |

---

## 3. Docker Core Architecture: Build, Ship, and Run

Docker operates around four core building blocks:

```
+------------+       docker build       +-----------------+       docker run       +---------------+
| Dockerfile |  --------------------->  | Container Image |  ------------------->  | Container     |
| (Recipe)   |                          | (Frozen Binary) |                        | (Running App) |
+------------+                          +-----------------+                        +---------------+
                                                ^
                                                | docker push / pull
                                                v
                                        +-------------------+
                                        | Container Registry|
                                        | (Docker Hub / ACR)|
                                        +-------------------+
```

1. **Dockerfile:** A text file containing sequential instructions telling Docker how to build your application image.
2. **Container Image:** An immutable, read-only package containing your source code, runtime, libraries, environment variables, and config files.
3. **Container:** A running, isolated process instantiated from a container image.
4. **Container Registry:** A centralized repository for sharing and storing container images (e.g., Docker Hub, Azure Container Registry `ACR`).

---

## 4. Dissecting a Production Dockerfile

Here is a real-world production Dockerfile for a Node.js web application with every directive explained:

```dockerfile
# 1. Base Image: Start with an official, lightweight Alpine Linux image with Node.js
FROM node:18-alpine

# 2. Working Directory: Set the internal folder where all app files will reside
WORKDIR /app

# 3. Cache Optimization: Copy package manifests first to leverage Docker layer caching
COPY package*.json ./

# 4. Install Dependencies: Run npm install inside the container during build
RUN npm install --production

# 5. Copy Application Code: Copy remaining source code files into the container
COPY . .

# 6. Expose Port: Document the network port the application listens on
EXPOSE 3000

# 7. Start Command: The default execution command when the container launches
CMD ["node", "server.js"]
```

### Essential Dockerfile Directives Cheat Sheet:
| Directive | Purpose | Example |
| :--- | :--- | :--- |
| `FROM` | Defines the parent base image. | `FROM ubuntu:22.04` or `FROM python:3.11-slim` |
| `WORKDIR` | Sets the working directory for subsequent instructions. | `WORKDIR /app` |
| `COPY` | Copies files from your local laptop into the container image. | `COPY . /app` |
| `RUN` | Executes build-time commands (installs packages). | `RUN apt update && apt install -y curl` |
| `ENV` | Sets persistent environment variables. | `ENV PORT=8080 NODE_ENV=production` |
| `EXPOSE` | Informs Docker which port the app listens on. | `EXPOSE 80` |
| `CMD` | The command executed when the container starts. | `CMD ["python", "app.py"]` |

---

## 5. Essential Docker CLI Command Reference

```bash
# 1. Build an image from a Dockerfile in the current directory
docker build -t my-web-app:v1.0 .

# 2. List all local container images
docker images

# 3. Run a container in detached mode (-d) with port forwarding (-p Host:Container)
docker run -d -p 80:3000 --name web-prod my-web-app:v1.0

# 4. List all currently RUNNING containers
docker ps

# 5. List ALL containers (including stopped/exited containers)
docker ps -a

# 6. View live real-time application logs streaming from the container
docker logs -f web-prod

# 7. Open an interactive bash/sh terminal INSIDE the running container
docker exec -it web-prod /bin/sh

# 8. Stop a running container
docker stop web-prod

# 9. Start a stopped container
docker start web-prod

# 10. Delete a container (Must be stopped first)
docker rm web-prod

# 11. Delete a container image
docker rmi my-web-app:v1.0
```

---

## 6. From Single Container to Orchestration: Why Kubernetes & AKS?

Running a single container on your laptop with `docker run` is straightforward. But when deploying an enterprise production application (like Amazon, Netflix, or Uber):
- What happens if a container crashes at 3:00 AM? Who restarts it?
- What happens when traffic surges by 1,000%? Who spins up 50 new container copies and load balances traffic across them?
- How do you update software with zero downtime (rolling updates)?
- How do 30 microservices discover and talk to each other securely?

**Kubernetes (K8s)** is the world's leading open-source container orchestration engine that automates container deployment, scaling, self-healing, and networking.

### Azure Kubernetes Service (AKS):
Deploying Kubernetes manually requires days of configuring master nodes, etcd databases, and certificates. **Azure Kubernetes Service (AKS)** makes this effortless:
- **Control Plane (Master Nodes):** Managed, patched, and provided **100% FREE** by Microsoft Azure.
- **Worker Nodes (Node Pools):** Azure Virtual Machines where your containers run. You only pay for the underlying worker VMs!

```
+--------------------------------------------------------------------------+
|                     AZURE KUBERNETES SERVICE (AKS)                       |
+--------------------------------------------------------------------------+
|                                                                          |
|  +--------------------------------------------------------------------+  |
|  |     MANAGED CONTROL PLANE (Master Nodes - 100% FREE & MANAGED)     |  |
|  |     [ API Server ]     [ etcd Database ]     [ Kube-Scheduler ]    |  |
|  +-----------------------------------+--------------------------------+  |
|                                      |                                   |
|                                      v                                   |
|  +--------------------------------------------------------------------+  |
|  |     WORKER NODE POOL (Virtual Machines - You only pay for these)   |  |
|  |                                                                    |  |
|  |   [ Worker Node 1 (VM) ]                 [ Worker Node 2 (VM) ]    |  |
|  |   +-----------------------+              +-----------------------+ |  |
|  |   | [Pod A]       [Pod B] |              | [Pod A]       [Pod C] | |  |
|  |   +-----------------------+              +-----------------------+ |  |
|  +--------------------------------------------------------------------+  |
+--------------------------------------------------------------------------+
```

---

## 7. Hands-on Lab: Deploying an Application on AKS

### Step 1: Create an AKS Cluster in Azure Portal
1. Search for **Kubernetes services** in the Azure Portal $\rightarrow$ click **+ Create** $\rightarrow$ **Create a Kubernetes cluster**.
2. **Basics Tab:**
   - **Resource group:** `rg-aks-demo`.
   - **Preset configuration:** **Dev/Test** (Cost-effective for learning).
   - **Cluster name:** `aks-demo-cluster`.
   - **Region:** `East US`.
3. **Node Pools Tab:**
   - Default node pool size: `Standard_B2s` (2 vCPU, 4 GiB RAM).
   - Node count: `1` (Keep at 1 to preserve free credits).
4. Click **Review + create** $\rightarrow$ **Create**. (Takes ~5 minutes).

---

### Step 2: Connect via Azure CLI & `kubectl`
```bash
# 1. Download cluster credentials into ~/.kube/config
az aks get-credentials --resource-group rg-aks-demo --name aks-demo-cluster

# 2. Verify worker nodes are Ready
kubectl get nodes
```

---

### Step 3: Deploy with Kubernetes Manifest (`deploy.yaml`)
Create `deploy.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-container
        image: mcr.microsoft.com/azuredocs/aci-helloworld:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: web-app
```

> **The Azure LoadBalancer Magic:**  
> Declaring `type: LoadBalancer` automatically instructs Azure to provision a real **Azure Public Load Balancer** with an external IP!

```bash
# Deploy to cluster
kubectl apply -f deploy.yaml

# Check public external IP
kubectl get service web-service --watch
```

Open your browser $\rightarrow$ navigate to the `EXTERNAL-IP` $\rightarrow$ Web page loads instantly!

---

## 8. Summary Checklist
- [x] Containers package code and dependencies, eliminating the "works on my machine" bug.
- [x] Containers share the host kernel, making them lightweight and bootable in milliseconds.
- [x] Docker images are immutable blueprints built from sequential Dockerfile instructions.
- [x] Container registries store and distribute images across local, cloud, and edge environments.
- [x] Kubernetes / AKS automates scaling, healing, and load balancing for container clusters.
