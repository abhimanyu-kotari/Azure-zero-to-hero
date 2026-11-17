# 🏗️ 05 - Modern Microservices: Azure Container Apps (ACA)

<div align="center">

![Microsoft Azure](https://img.shields.io/badge/Azure_Container_Apps-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Scale to Zero](https://img.shields.io/badge/Scale_to_Zero-$0.00_Idle-brightgreen?style=for-the-badge)
![Envoy Proxy](https://img.shields.io/badge/Envoy-Ingress_%26_SSL-purple?style=for-the-badge&logo=envoyproxy&logoColor=white)
![KEDA](https://img.shields.io/badge/KEDA-Event_Autoscaling-red?style=for-the-badge)
![Canary](https://img.shields.io/badge/Traffic_Splitting-Canary_80%2F20-blue?style=for-the-badge)

</div>

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:29:29` to `04:38:36` (Serverless Microservices & Traffic Splitting)

---

## 🏗️ 1. Why Azure Container Apps (ACA)? The Microservices Dilemma

In the previous modules, we explored two container compute options:
- **Azure Container Instances (ACI):** Perfect for running a single isolated container, but lacks built-in HTTP load balancing, traffic splitting, service discovery, and automatic scaling.
- **Azure Kubernetes Service (AKS):** Full enterprise Kubernetes orchestration, but requires deep DevOps expertise, complex YAML files, and continuous hourly charges for running worker VMs.

Modern software companies (e.g., e-commerce, banking, food delivery) build **microservice architectures** consisting of multiple interconnected APIs:
- `Frontend Web App`
- `Product Catalog API`
- `Shopping Cart Service`
- `Payment Processing Service`

Developers want the benefits of Kubernetes (automatic scaling, self-healing, rolling updates, ingress routing) **without having to manage a Kubernetes cluster**.

**Azure Container Apps (ACA) is Microsoft's answer:**
> **"All the architectural superpowers of Kubernetes and microservices, wrapped in a pure serverless experience where you never manage a cluster."**

```
+---------------------------------------------------------------------------------------------------+
|                                 THE AZURE CONTAINER COMPUTE SPECTRUM                              |
+---------------------------------------------------------------------------------------------------+
|                                                                                                   |
|  [ Azure Container Instances (ACI) ]   --->  [ Azure Container Apps (ACA) ]  ---> [ Azure AKS ]   |
|  - Single Container Task                     - Microservices Architecture         - Enterprise K8s|
|  - No Auto-scaling                           - Scale-to-Zero ($0.00 when idle!)   - Full K8s API  |
|  - Manual networking                         - Built-in Ingress & Envoy Proxy     - Custom Helm   |
|  - Simplest to use                           - Canary Traffic Splitting (80/20)   - Most Complex  |
|                                                                                                   |
+---------------------------------------------------------------------------------------------------+
```

---

## 2. The Open-Source Engine Powering Azure Container Apps

Behind the scenes, Microsoft engineered Azure Container Apps using battle-tested open-source cloud technologies:

```
+--------------------------------------------------------------------------+
|                     INSIDE AZURE CONTAINER APPS                          |
+--------------------------------------------------------------------------+
|                                                                          |
|   +------------------------------------------------------------------+   |
|   |   BUILT-IN INGRESS (Envoy Proxy)                                 |   |
|   |   - Automatic HTTPS Termination & Free Auto-Renewing SSL         |   |
|   |   - Traffic Splitting (e.g., 80% to v1, 20% to v2 Canary)        |   |
|   +---------------------------------+--------------------------------+   |
|                                     |                                    |
|                                     v                                    |
|   +------------------------------------------------------------------+   |
|   |   APPLICATION CONTAINERS (Your Microservice Code)                |   |
|   |   - Immutable Revisions & Instant Blue/Green Rollbacks           |   |
|   +---------------------------------+--------------------------------+   |
|                                     |                                    |
|                                     v                                    |
|   +------------------------------------------------------------------+   |
|   |   AUTOSCALING ENGINE (KEDA - Event-Driven Autoscaling)           |   |
|   |   - Scale on HTTP requests, CPU, RAM, or Azure Queue messages    |   |
|   |   - SCALE TO ZERO (Idle apps consume $0.00!)                     |   |
|   +------------------------------------------------------------------+   |
|                                                                          |
+--------------------------------------------------------------------------+
```

### Core Superpowers of ACA:
1. **Scale-to-Zero (Huge Cost Saver):**  
   If your application receives no traffic at 3:00 AM, ACA automatically terminates running container instances down to `0`. **Your compute bill drops to $0.00!** The moment a user makes an HTTP request, Envoy proxy intercepts it and ACA spins up a container in seconds to process the request.
2. **Canary Deployments & Traffic Splitting:**  
   Safely test new software versions in production by routing `80%` of user requests to your stable `v1` version and `20%` of requests to your new `v2` version.
3. **Immutable Revisions & Instant Rollback:**  
   Every time you deploy updated code or environment variables, ACA creates an immutable revision. If a bug is discovered in `v2`, you can roll back to `v1` with a single click.
4. **Dapr (Distributed Application Runtime) Integration:**  
   Built-in microservices support for pub/sub messaging, state management, and secure service-to-service discovery.

---

## 3. Comparison Matrix: ACI vs. ACA vs. AKS

| Feature | Azure Container Instances (ACI) | Azure Container Apps (ACA) | Azure Kubernetes Service (AKS) |
| :--- | :--- | :--- | :--- |
| **Primary Use Case** | Batch tasks, single-run scripts | **Web apps, APIs, microservices** | Complex enterprise K8s systems |
| **Scale to Zero** | No (Must manually stop container) | **Yes (Automatic when idle = $0)**| Complex (Requires virtual nodes) |
| **Built-in HTTPS Ingress**| No (Raw TCP ports only) | **Yes (Free auto-renewing SSL)** | Requires Nginx/Traefik Ingress |
| **Traffic Splitting** | No | **Yes (Built-in slider percentage)**| Requires Service Mesh (Istio/Linkerd) |
| **K8s Knowledge Needed** | None | **None** | High (YAML, Helm, RBAC, etcd) |

---

## 4. Hands-on Lab: Deploying a Container App in the Portal

In this lab from the video, M Prashant demonstrates deploying a containerized application with built-in HTTPS ingress:

### Step 1: Create the Container App in Azure Portal
1. Open the [Azure Portal](https://portal.azure.com).
2. Search for **Container Apps** in the top search bar $\rightarrow$ click **+ Create**.

#### Tab 1: Basics
- **Subscription:** Select your subscription.
- **Resource group:** Click *Create new* $\rightarrow$ `rg-aca-demo`.
- **Container App name:** `my-microservice-app`.
- **Region:** `East US`.
- **Container Apps Environment:** Click *Create new*:
  - Environment name: `cae-production-environment`.
  - Zone redundancy: *Disabled* (to save cost for learning).
  - Click **Create**.

---

#### Tab 2: Container Settings
- **Image source:** Select **Quickstart image**.
- **Image:** `mcr.microsoft.com/k8se/quickstart:latest` (Linux).
- **CPU:** `0.25 vCPU` (Ultra-low cost).
- **Memory:** `0.5 GiB`.

---

#### Tab 3: Ingress Configuration
- **Ingress:** Check `[x] Enabled`.
- **Ingress traffic:** Select **Accepting traffic from anywhere** (Public internet access).
- **Insecure connections:** Leave unchecked (Forces encrypted HTTPS).
- **Target port:** Enter `80` (The port the container listens on).
- Click **Review + create** $\rightarrow$ click **Create**. (Takes ~2 minutes).

---

### Step 2: Verify the Live HTTPS Endpoint
1. Once deployment succeeds, click **Go to resource**.
2. On the **Overview** page, locate the **Application Url** (e.g., `https://my-microservice-app.proudforest-1234.eastus.azurecontainerapps.io`).
3. Click the URL: The application opens immediately over secure HTTPS with an official Microsoft SSL certificate!

---

### Step 3: Canary Deployments via Traffic Splitting
To demonstrate safe production releases without downtime:

1. In the left navigation menu under **Application**, click **Revisions and replicas**.
2. Click **Create new revision**:
   - Provide an updated container image tag (or add an environment variable `VERSION=v2.0`).
   - Click **Create**.
3. In the Revisions window, click **Traffic splitting**:
   - Set `my-microservice-app--rev1`: `80%`
   - Set `my-microservice-app--rev2`: `20%`
   - Click **Save**.
4. Test the traffic distribution from your terminal using a curl loop:
   ```bash
   for i in {1..10}; do curl -s https://my-microservice-app.proudforest-1234.eastus.azurecontainerapps.io | grep "Welcome"; done
   ```
   Envoy proxy dynamically routes 8 out of 10 requests to Revision 1, and 2 out of 10 requests to Revision 2 with **zero dropped connections or server reboots**!

---

## 5. Troubleshooting Common ACA Issues

| Problem | Root Cause | Exact Resolution |
| :--- | :--- | :--- |
| First request after idle period takes 5 to 8 seconds. | **Cold start delay** while waking up from 0 replicas. | Set **Min replicas = 1** in the Scale tab if your production SLA requires zero cold-start delay. |
| Ingress URL displays `HTTP 502 Bad Gateway`. | Port mismatch between ACA ingress and container app. | Verify that the **Target port** in Ingress settings matches the internal port the web framework listens on (e.g., 80, 3000, 8080). |
| Image pull failure from private Azure Container Registry (ACR). | Missing identity authorization. | Go to **Authentication** in Container App settings and link your Azure Managed Identity with `AcrPull` permissions. |

---

## 6. Summary Checklist
- [x] ACA provides serverless microservices without exposing Kubernetes complexity.
- [x] Built on battle-tested open-source tools: Kubernetes, KEDA, and Envoy Proxy.
- [x] Scale-to-Zero eliminates compute billing when application traffic drops to zero.
- [x] Native HTTPS Ingress provides free auto-renewing SSL certificates.
- [x] Traffic splitting enables zero-downtime canary releases and instant revision rollbacks.
