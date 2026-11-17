# 07 - Docker & Azure Kubernetes Service (AKS)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `03:47:55` to `04:23:32`

---

## 1. Why Do We Need Container Orchestration?

In module 03, we learned that Docker containers package applications so they run reliably anywhere. Running **one** container on your laptop with `docker run` is easy.

**However, what happens in a real enterprise with 100+ microservices running across 20 servers?**
- What happens when a container crashes at 2:00 AM? Who restarts it?
- If website traffic doubles, who launches 10 new container copies and configures the load balancer?
- When you release Version 2 of your code, how do you upgrade containers without dropping active customer connections?
- If a physical server fails, who moves the running containers to a healthy server?

**Manual Docker commands cannot handle this.** You need a **Container Orchestration Engine**.

---

## 2. What is Kubernetes (K8s)?

**Kubernetes** (often abbreviated as **K8s**) is an open-source container orchestration platform originally developed by Google. It automates the deployment, scaling, healing, and management of containerized applications.

### The Symphony Orchestra Analogy:
- Each individual container is like an instrument player (a violinist, drummer, flutist). They can play their own music well.
- **Kubernetes is the Maestro / Conductor:** It coordinates all the musicians together, tells them when to play louder (scale up), steps in when someone drops their instrument (auto-heals a crashed container), and ensures a harmonious performance.

---

## 3. What is Azure Kubernetes Service (AKS)?

Running raw Kubernetes yourself requires deploying and managing master nodes, configuring etcd clustering, generating TLS certificates, and applying operating system patches—a massive operational headache.

**Azure Kubernetes Service (AKS)** is Microsoft's fully managed Kubernetes service:

```
+---------------------------------------------------------------------------------+
|                         AZURE MANAGED CONTROL PLANE (FREE)                      |
|                                                                                 |
|   +--------------------+  +--------------------+  +--------------------------+  |
|   |   kube-apiserver   |  |   kube-scheduler   |  |  kube-controller-manager |  |
|   +--------------------+  +--------------------+  +--------------------------+  |
|                      +-------------------------------+                          |
|                      |             etcd              |                          |
|                      +-------------------------------+                          |
|                                                                                 |
|   * Microsoft handles 100% of master nodes, backups, security patches & uptime  |
+----------------------------------------+----------------------------------------+
                                         | Secure Tunnel
                                         v
+---------------------------------------------------------------------------------+
|                        CUSTOMER WORKER NODES (DATA PLANE)                       |
|                                                                                 |
|   +----------------------------------+    +----------------------------------+  |
|   |           Worker Node 1          |    |           Worker Node 2          |  |
|   |         (Azure VM / VMSS)        |    |         (Azure VM / VMSS)        |  |
|   |                                  |    |                                  |  |
|   |   [ Pod 1 ]          [ Pod 2 ]   |    |   [ Pod 3 ]          [ Pod 4 ]   |  |
|   |  (Container)        (Container)  |    |  (Container)        (Container)  |  |
|   +----------------------------------+    +----------------------------------+  |
|                                                                                 |
|   * You only pay for the worker node VMs running your actual containers!        |
+---------------------------------------------------------------------------------+
```

### The Best Part of AKS:
Microsoft provides the entire **Kubernetes Control Plane (Master Nodes) completely free of cost**! You only pay for the worker node VMs that execute your application containers.

---

## 4. Fundamental Kubernetes Concepts Every Student Must Know

Kubernetes uses declarative YAML manifests to describe what your infrastructure should look like:

### 1. Pod (The Smallest Building Block)
- In Kubernetes, you don't run containers directly on nodes. Instead, you run **Pods**.
- A Pod is a wrapper around one (or more closely related) containers.
- Pods have their own unique private IP address and share storage volumes.

### 2. Deployment (Desired State & Self-Healing)
- A **Deployment** manages the creation and scaling of Pods.
- You tell the Deployment: *"I want 3 replicas of the Nginx web pod."*
- If a node crashes and destroys Pod 2, the Deployment controller notices that only 2 pods are running and **instantly launches a replacement Pod on another node**.

### 3. Service (Networking & Load Balancing)
- Pods are ephemeral (temporary). When a pod dies and is recreated, it gets a brand-new IP address.
- A **Service** provides a stable, permanent IP address and DNS name in front of a group of pods.
- **Service Types:**
  - **`ClusterIP` (Default):** Accessible only inside the Kubernetes cluster.
  - **`NodePort`:** Exposes the service on a static port on each worker node's IP.
  - **`LoadBalancer`:** Automatically provisions a real **Azure Public Load Balancer** with a public IP routing internet traffic into your pods!

---

## 5. Hands-on Walkthrough: Deploying Your First App on AKS

In the video, M Prashant demonstrates creating an AKS cluster and deploying an app via `kubectl`:

### Step 1: Create an AKS Cluster
1. In the Azure Portal, search for **Kubernetes services** -> Click **Create** -> **Create a Kubernetes cluster**.
2. **Basics:**
   - Resource Group: `rg-aks-demo`.
   - Cluster name: `my-student-aks`.
   - Region: `East US`.
   - Kubernetes version: Keep default (e.g., 1.28 or latest stable).
3. **Node Pools:**
   - Primary node pool size: `Standard_B2s` (affordable for practice).
   - Node count: `2`.
4. Click **Review + Create**, then click **Create** (takes ~4 to 7 minutes to provision).

### Step 2: Connect to AKS via Terminal (`kubectl`)
Open Azure Cloud Shell (or your local terminal with Azure CLI installed):

```bash
# 1. Download cluster credentials into your local kubeconfig file
az aks get-credentials --resource-group rg-aks-demo --name my-student-aks

# 2. Verify connection by listing your worker nodes
kubectl get nodes
```

### Step 3: Deploy an Application (Nginx Web Server)
```bash
# 1. Create a deployment running 2 replicas of Nginx
kubectl create deployment my-web-app --image=nginx --replicas=2

# 2. Check the running pods
kubectl get pods -o wide
```

### Step 4: Expose the App to the Internet
```bash
# Expose the deployment using an Azure Public Load Balancer on Port 80
kubectl expose deployment my-web-app --type=LoadBalancer --port=80

# Check the service status and wait for the EXTERNAL-IP to be assigned by Azure
kubectl get svc my-web-app --watch
```

Output:
```
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
my-web-app   LoadBalancer   10.0.145.22    20.120.89.14     80:31254/TCP   45s
```

Copy the **EXTERNAL-IP** (`20.120.89.14`) and paste it into your browser. You will see the **Nginx welcome page** being served directly by your Kubernetes pods on Azure!

---

## 6. Summary Checklist
- [x] Kubernetes automates scaling, healing, rolling updates, and networking for containers.
- [x] AKS provides a fully managed, free Kubernetes control plane.
- [x] Pods are the smallest deployable units; Deployments manage replica counts and self-healing.
- [x] Kubernetes Services of type `LoadBalancer` automatically create Azure Public Load Balancers.
- [x] Use `kubectl` CLI to inspect, deploy, scale, and manage your cluster resources.
