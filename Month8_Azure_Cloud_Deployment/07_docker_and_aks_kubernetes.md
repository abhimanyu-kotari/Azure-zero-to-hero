# 07 - Containers & Orchestration: Docker & Azure Kubernetes Service (AKS)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `03:47:55` to `04:23:32`

---

## 1. Virtual Machines vs. Containers

To understand modern cloud development, you must understand why the software industry transitioned from Virtual Machines to **Containers**:

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

| Feature | Virtual Machine (VM) | Container (Docker) |
| :--- | :--- | :--- |
| **Virtualization Level** | Hardware level (Virtualizes CPU, RAM, NIC) | OS level (Shares the host Linux kernel) |
| **Operating System** | Requires a complete Guest OS (Windows/Linux) | No Guest OS; packages only app code + libraries |
| **Startup Time** | Minutes (Booting complete kernel) | **Milliseconds to seconds** |
| **Size on Disk** | 5 GB to 50 GB per VM | **50 MB to 500 MB** per image |
| **Resource Overhead** | High (Every VM consumes dedicated RAM for OS) | Extremely lightweight and efficient |

---

## 2. Docker Fundamentals & Architecture

**Docker** is the industry-standard platform for building, running, and managing containers.

### Core Terminology:
1. **Dockerfile:** A text file containing sequential instructions used to build a container image (e.g., base image, install dependencies, copy code, expose port).
2. **Container Image:** A packaged, read-only template snapshot containing your application code, runtime, libraries, and environment variables.
3. **Container:** A running, runnable instance of an image.
4. **Container Registry:** A repository for storing and distributing images (e.g., Docker Hub, Azure Container Registry `ACR`).

```
+------------+       docker build       +-----------------+       docker run       +---------------+
| Dockerfile |  --------------------->  | Container Image |  ------------------->  | Container     |
| (Recipe)   |                          | (Frozen Binary) |                        | (Running App) |
+------------+                          +-----------------+                        +---------------+
                                                ^
                                                | docker push / pull
                                                v
                                        +-------------------+
                                        | Azure Container   |
                                        | Registry (ACR)    |
                                        +-------------------+
```

---

## 3. Why Do We Need Kubernetes?

Running one Docker container on your laptop using `docker run` is straightforward. But in enterprise production, you must manage:
- Hundreds of microservices
- Automatic rollouts and zero-downtime rollbacks
- Auto-scaling up during flash sales and down during the night
- Self-healing (restarting crashed containers immediately)
- Service discovery and internal network routing

**Kubernetes (K8s)** is an open-source container orchestration system that automates the deployment, scaling, and management of containerized applications.

---

## 4. What is Azure Kubernetes Service (AKS)?

Deploying Kubernetes manually on bare-metal or raw VMs is notoriously complex: you must configure etcd clusters, TLS certificates, API servers, and control plane networking.

**Azure Kubernetes Service (AKS)** is Microsoft's fully managed Kubernetes offering:
- **Control Plane (Master Node):** Completely managed, secured, patched, and provided **FREE OF CHARGE** by Microsoft Azure.
- **Worker Nodes (Node Pools):** Azure provisioned VMs where your container pods run. You only pay for these underlying VM instances!

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

## 5. Hands-on Lab: Deploying an Application to AKS

In this walkthrough, we create an AKS cluster, connect using `kubectl`, and deploy a web application accessible via a public IP.

### Step 1: Create an AKS Cluster
1. Open the [Azure Portal](https://portal.azure.com).
2. Search for **Kubernetes services** and click **+ Create** -> **Create a Kubernetes cluster**.
3. **Basics Tab:**
   - **Subscription:** Your subscription.
   - **Resource group:** `rg-aks-demo`.
   - **Cluster preset configuration:** **Dev/Test** (Optimized for low-cost learning).
   - **Kubernetes cluster name:** `aks-cluster-demo`.
   - **Region:** `East US`.
   - **Availability zones:** None (to save cost for learning).
   - **Kubernetes version:** Leave default recommended.
4. **Node Pools Tab:**
   - Node size: Select `Standard_B2s` (2 vCPUs, 4 GiB RAM).
   - Scale method: **Manual**.
   - Node count: `1` or `2` (Keep at 1 for free tier / minimal credit use).
5. **Networking Tab:**
   - Network configuration: **Kubenet** (Basic) or **Azure CNI**.
6. Click **Review + create** -> **Create**.
7. AKS cluster provisioning takes approximately 5–8 minutes.

---

### Step 2: Connect to AKS via Azure CLI & kubectl
Run these commands in Azure Cloud Shell (or your local terminal with Azure CLI installed):

```bash
# 1. Install kubectl CLI if not already present
az aks install-cli

# 2. Download the Kubernetes credentials and configure your kubeconfig
az aks get-credentials --resource-group rg-aks-demo --name aks-cluster-demo

# 3. Verify connection to the cluster
kubectl get nodes
```
**Expected Output:**
```
NAME                                STATUS   ROLES   AGE   VERSION
aks-nodepool1-12345678-vmss000000   Ready    agent   5m    v1.28.3
```

---

### Step 3: Deploy an Application using Kubernetes Manifest (YAML)
Create a file named `azure-vote.yaml` defining a **Deployment** (the pods) and a **Service** (the public load balancer):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-deployment
  labels:
    app: web-app
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
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "250m"
            memory: "256Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: web-app
```

> **The Magic of `type: LoadBalancer`:** When you specify `type: LoadBalancer`, Azure Kubernetes Service automatically talks to the Azure Resource Manager API and provisions a real **Azure Public Load Balancer** with a dedicated public IP address!

---

### Step 4: Apply the Manifest & Verify
```bash
# Apply the configuration to the cluster
kubectl apply -f azure-vote.yaml

# Check the running pods
kubectl get pods

# Check the service and wait for the External-IP to be assigned
kubectl get service web-app-service --watch
```

**Output:**
```
NAME               TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
web-app-service    LoadBalancer   10.0.145.22    20.84.112.95    80:31234/TCP   2m
```

Open your browser and visit `http://20.84.112.95`:
You will see the sample application running across your Kubernetes cluster!

---

### Step 5: Scaling the Application in Real-Time
To increase capacity from 2 pods to 5 pods instantly:
```bash
kubectl scale deployment web-app-deployment --replicas=5
kubectl get pods
```
Within 3 seconds, 5 container instances are running and receiving traffic!

---

### Step 6: Critical Cleanup (Save Money!)
AKS clusters run continuously and can consume your budget if left running:
```bash
# Delete the entire Resource Group to remove the cluster and all attached disks/load balancers
az group delete --name rg-aks-demo --yes --no-wait
```

---

## 6. Summary Checklist
- [x] Containers package code and dependencies without the heavy overhead of a Guest OS.
- [x] Kubernetes automates deployment, scaling, healing, and discovery for container clusters.
- [x] AKS provides a 100% free, fully managed control plane; you only pay for the worker node VMs.
- [x] Declaring `type: LoadBalancer` in a Kubernetes service automatically provisions an Azure Public Load Balancer.
- [x] Always delete or stop AKS clusters when not in use to avoid ongoing VM node charges.
