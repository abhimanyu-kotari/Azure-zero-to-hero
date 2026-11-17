# 07 - Containers & Orchestration: Docker & Azure Kubernetes Service (AKS)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `03:47:55` to `04:23:32`

---

## 1. Virtual Machines vs. Containers

To understand modern cloud-native development, you must understand why the software industry shifted from Virtual Machines to **Containers**:

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

| Metric | Virtual Machine (VM) | Container (Docker) |
| :--- | :--- | :--- |
| **Virtualization Level** | Hardware level (Virtualizes CPU, RAM, NIC) | OS level (Shares the host Linux kernel) |
| **Operating System** | Requires a complete Guest OS per VM | No Guest OS; packages only app code + libraries |
| **Bootup Time** | Minutes (Booting complete kernel) | **Milliseconds to seconds** |
| **Size on Disk** | 5 GB to 50 GB per VM image | **50 MB to 500 MB** per container image |
| **Memory Overhead** | High (Every VM reserves dedicated RAM for OS) | Extremely lightweight and efficient |
| **Portability** | Hypervisor-dependent | Runs identically anywhere Docker is installed |

---

## 2. Docker Fundamentals: Build, Ship, and Run

**Docker** is the open-source platform that popularized containerization.

### Core Terminology:
1. **Dockerfile:** A plain text configuration file containing sequential commands to assemble a container image.
2. **Container Image:** A frozen, read-only snapshot containing your source code, runtime, libraries, environment variables, and config files.
3. **Container:** A running, isolated instance of an image.
4. **Container Registry:** A cloud repository for storing and versioning container images (e.g., Docker Hub, Azure Container Registry `ACR`).

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

### Sample Node.js Dockerfile:
```dockerfile
# 1. Base image with Node.js pre-installed
FROM node:18-alpine

# 2. Set working directory inside container
WORKDIR /app

# 3. Copy package manifests and install dependencies
COPY package*.json ./
RUN npm install --production

# 4. Copy application source code
COPY . .

# 5. Expose web port
EXPOSE 3000

# 6. Command to start the application
CMD ["node", "server.js"]
```

```bash
# Build the container image
docker build -t my-web-app:v1 .

# Run the container locally on port 80
docker run -d -p 80:3000 --name web-container my-web-app:v1
```

---

## 3. Why Do We Need Kubernetes?

Running a single container on your laptop using `docker run` is simple. But in enterprise production, you must manage:
- Hundreds of microservice containers communicating across multiple servers.
- Automatic scaling up during spikes and down during quiet hours.
- Self-healing: if a container crashes at 3 AM, automatically restarting it in seconds.
- Zero-downtime rolling updates and automated rollbacks.
- Service discovery and internal load balancing.

**Kubernetes (K8s)** is an open-source container orchestration engine that automates the deployment, scaling, and operational management of containerized workloads.

---

## 4. What is Azure Kubernetes Service (AKS)?

Deploying Kubernetes manually from scratch requires configuring etcd databases, managing TLS certificates, and provisioning API servers.

**Azure Kubernetes Service (AKS)** is Microsoft’s fully managed Kubernetes offering:
- **Control Plane (Master Node):** Completely managed, monitored, patched, and provided **100% FREE OF CHARGE** by Microsoft Azure.
- **Worker Nodes (Node Pools):** Azure-managed Virtual Machine Scale Sets where your actual container pods run. You only pay for these underlying VM instances!

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

In this lab from the video, M Prashant demonstrates provisioning an AKS cluster and deploying an application accessible over a public IP.

### Step 1: Create an AKS Cluster in Azure Portal
1. Search for **Kubernetes services** in the Azure Portal and click **+ Create** -> **Create a Kubernetes cluster**.
2. **Basics Tab:**
   - **Subscription:** Select your subscription.
   - **Resource group:** `rg-aks-demo`.
   - **Cluster preset configuration:** **Dev/Test** (Cheapest configuration for learning).
   - **Kubernetes cluster name:** `aks-cluster-demo`.
   - **Region:** `East US`.
   - **Availability zones:** None (to save cost for learning).
   - **Kubernetes version:** Leave default recommended.
3. **Node Pools Tab:**
   - Click on the default node pool:
     - Node size: `Standard_B2s` (2 vCPUs, 4 GiB RAM).
     - Scale method: **Manual**.
     - Node count: `1` (Keep at 1 for free trial / minimal credit burn).
4. **Networking Tab:**
   - Network configuration: **Kubenet** (Basic).
5. Click **Review + create** -> **Create**.
6. Cluster provisioning takes approximately 5–8 minutes.

---

### Step 2: Connect to the Cluster via Azure CLI & kubectl
Run these commands in Azure Cloud Shell or your local terminal with Azure CLI installed:

```bash
# 1. Install kubectl CLI if not present
az aks install-cli

# 2. Download the cluster credentials into your ~/.kube/config file
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

### Step 3: Write the Kubernetes Manifest (`app-deploy.yaml`)
Create a file named `app-deploy.yaml` defining a **Deployment** (the pods) and a **Service** (the public load balancer):

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

> **The Power of `type: LoadBalancer`:**  
> When you declare `type: LoadBalancer`, Azure's cloud controller manager automatically talks to the Azure Resource Manager API and provisions a real **Azure Public Load Balancer** with a dedicated public IP address!

---

### Step 4: Apply Manifest & Verify Application
```bash
# Apply the configuration
kubectl apply -f app-deploy.yaml

# Check pods status
kubectl get pods

# Check service and wait for External-IP to be allocated
kubectl get service web-app-service --watch
```

**Output:**
```
NAME               TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
web-app-service    LoadBalancer   10.0.145.22    20.84.112.95    80:31234/TCP   2m
```

Open your browser and visit: `http://20.84.112.95`  
The sample container application loads instantly!

---

### Step 5: Instant Pod Scaling
To scale up your application from 2 pods to 5 pods in seconds:
```bash
kubectl scale deployment web-app-deployment --replicas=5
kubectl get pods
```
All 5 pods are running in parallel, with traffic load-balanced automatically!

---

### Step 6: Critical Cost Cleanup
AKS clusters run continuously. To avoid burning subscription credits:
```bash
az group delete --name rg-aks-demo --yes --no-wait
```

---

## 6. Troubleshooting Common AKS Issues

| Problem | Cause | Resolution |
| :--- | :--- | :--- |
| `EXTERNAL-IP` stays `<pending>` indefinitely. | Azure Load Balancer quota reached or subnet exhausted. | Check Azure subscription quota for Public IPs. Ensure your node pool subnet has free IPs. |
| Pod status displays `CrashLoopBackOff`. | Application inside container crashed on startup. | Run `kubectl logs <pod-name>` and `kubectl describe pod <pod-name>` to view container error logs. |
| `az aks get-credentials` fails with authorization error. | Missing Azure RBAC permissions on AKS resource. | Ensure your Entra ID user has `Azure Kubernetes Service Cluster Admin Role`. |

---

## 7. Summary Checklist
- [x] Containers package code and dependencies without the heavy overhead of a Guest OS.
- [x] Kubernetes automates deployment, scaling, self-healing, and networking for container clusters.
- [x] AKS provides a 100% free, fully managed control plane; you only pay for worker node VMs.
- [x] Declaring `type: LoadBalancer` in a Kubernetes service automatically provisions an Azure Public Load Balancer.
- [x] Always delete or stop AKS clusters when not in use to avoid ongoing VM charges.
