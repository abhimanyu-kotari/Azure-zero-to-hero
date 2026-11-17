# 09 - Modern Microservices: Azure Container Apps (ACA)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:29:29` to `04:38:36`

---

## 1. Why Azure Container Apps (ACA)?

In the previous modules, we explored two container hosting options in Azure:
1. **Azure Container Instances (ACI):** Extremely simple, but lacks ingress routing, traffic splitting, service discovery, and auto-scaling.
2. **Azure Kubernetes Service (AKS):** Extremely powerful, but requires deep Kubernetes expertise, complex YAML, helm charts, and dedicated node pool costs.

**Azure Container Apps (ACA)** fills the sweet spot:
> **"All the power of Kubernetes and microservices, with zero cluster management overhead."**

```
+--------------------------------------------------------------------------+
|                  THE AZURE CONTAINER SPECTRUM                            |
+--------------------------------------------------------------------------+
|  [ Azure ACI ]              [ Azure Container Apps ]      [ Azure AKS ]  |
|  - Single Container         - Microservices               - Enterprise K8s|
|  - No Auto-scaling          - Scale-to-Zero ($0 idle!)    - Custom Helm   |
|  - Manual networking        - Built-in Ingress & Envoy    - Full K8s API  |
|  (Simplest)                 - Traffic Splitting (Canary)  (Most Complex)  |
|                             - Serverless (KEDA + Envoy)                  |
+--------------------------------------------------------------------------+
```

---

## 2. The Open-Source Tech Stack Powering ACA

Behind the scenes, Microsoft engineered Azure Container Apps using industry-standard open-source technologies:

```
+--------------------------------------------------------------------------+
|                     INSIDE AZURE CONTAINER APPS                          |
+--------------------------------------------------------------------------+
|                                                                          |
|   +------------------------------------------------------------------+   |
|   |   BUILT-IN INGRESS (Envoy Proxy)                                 |   |
|   |   - Automatic HTTPS Termination & Free SSL                       |   |
|   |   - Traffic Splitting (e.g., 80% to v1, 20% to v2 Canary)        |   |
|   +---------------------------------+--------------------------------+   |
|                                     |                                    |
|                                     v                                    |
|   +------------------------------------------------------------------+   |
|   |   APPLICATION CONTAINERS (Your Microservice Code)                |   |
|   |   - Immutable Revisions & Blue/Green Deployments                 |   |
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

### Core Architecture Capabilities:
1. **Scale-to-Zero:** If no users visit your API at 3:00 AM, running instances scale down to `0`. You are billed **$0.00**. When an incoming request hits Envoy, ACA spins up a container in seconds!
2. **Traffic Splitting (Canary Releases):** Easily route 80% of traffic to your stable `v1` revision and 20% to a new `v2` revision to test new code safely.
3. **Revisions Management:** Every code or config update creates an immutable revision, enabling one-click instant rollbacks if issues occur.
4. **Dapr (Distributed Application Runtime) Integration:** Built-in support for pub/sub messaging, state management, and service-to-service discovery.

---

## 3. Comparing ACI, ACA, and AKS

| Feature | ACI | ACA | AKS |
| :--- | :--- | :--- | :--- |
| **Primary Use Case** | Batch tasks, isolated runs | Web apps, APIs, microservices | Complex enterprise orchestrations |
| **Scale to Zero** | No (You manually stop it) | **Yes (Automatic when idle)** | Complex (requires virtual nodes/KEDA) |
| **Built-in HTTPS Ingress**| No | **Yes (Free automatic SSL)** | Requires Ingress Controller (Nginx/Traefik) |
| **Traffic Splitting** | No | **Yes (Slider-based % routing)**| Requires Service Mesh (Istio/Linkerd) |
| **Kubernetes Knowledge Needed** | None | **None** | High |

---

## 4. Hands-on Lab: Deploying a Container App in the Portal

### Step 1: Create the Container App
1. Open the [Azure Portal](https://portal.azure.com).
2. Search for **Container Apps** and click **+ Create**.

#### Tab 1: Basics
- **Subscription:** Select your subscription.
- **Resource group:** `rg-aca-demo`.
- **Container App name:** `my-microservice-app`.
- **Region:** `East US`.
- **Container Apps Environment:** Click *Create new*:
  - Environment name: `cae-production-environment`.
  - Zone redundancy: *Disabled* (for dev/test).
  - Click **Create**.

---

#### Tab 2: Container Settings
- **Image source:** **Quickstart image**.
- **Image:** `mcr.microsoft.com/k8se/quickstart:latest`.
- **CPU:** `0.25 vCPU` (Ultra-low cost).
- **Memory:** `0.5 GiB`.

---

#### Tab 3: Ingress Configuration
- **Ingress:** `Enabled`.
- **Ingress traffic:** `Accepting traffic from anywhere` (Public internet).
- **Insecure connections:** Leave unchecked (Forces HTTPS).
- **Target port:** `80` (The port the container listens on).
- Click **Review + create** -> **Create**.

---

### Step 2: Verify the Live Endpoint
1. Once deployed, open your Container App **Overview** page.
2. Copy the **Application Url** (e.g., `https://my-microservice-app.victoriouscliff-12345.eastus.azurecontainerapps.io`).
3. Open the link in a browser. The application loads instantly over secure HTTPS!

---

### Step 3: Canary Deployments via Traffic Splitting
1. In the left navigation menu under **Application**, click **Revisions and replicas**.
2. Click **Create new revision**:
   - Provide an updated container image (or update environment variables).
   - Click **Save**.
3. Once both revisions are listed:
   - Click **Traffic splitting**.
   - Assign `80%` traffic to `Revision 1`.
   - Assign `20%` traffic to `Revision 2`.
   - Click **Save**.
4. Test by running a terminal loop:
   ```bash
   for i in {1..10}; do curl -s https://my-microservice-app.victoriouscliff-12345.eastus.azurecontainerapps.io | grep "Welcome"; done
   ```
   Envoy splits traffic according to your exact percentages without any server restarts or dropped requests!

---

## 5. Troubleshooting Common ACA Issues

| Problem | Cause | Resolution |
| :--- | :--- | :--- |
| First request after idle period takes 5–8 seconds. | Cold start delay while scaling up from 0 replicas. | Set **Min replicas = 1** in the Scaling tab if zero cold start latency is required for production. |
| Ingress URL returns `HTTP 502 Bad Gateway`. | Container failed startup probe or target port mismatch. | Verify the **Target port** in Ingress settings matches the port your container app listens on (e.g., 80, 3000, 8080). |
| Image pull failure from private ACR. | Missing Managed Identity or ACR admin credentials. | Go to **Authentication** in Container App settings and link your Azure Container Registry credentials. |

---

## 6. Summary Checklist
- [x] ACA delivers serverless microservices without exposing Kubernetes complexity.
- [x] Powered by enterprise open-source tools: Kubernetes, KEDA, and Envoy Proxy.
- [x] Automatic scale-to-zero completely eliminates compute billing when apps are idle.
- [x] Includes out-of-the-box HTTPS ingress and percentage-based traffic splitting for canary releases.
