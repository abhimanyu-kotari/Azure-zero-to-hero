# 09 - Modern Microservices: Azure Container Apps (ACA)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:29:29` to `04:38:36`

---

## 1. Why Azure Container Apps (ACA)?

In the previous modules, we saw two extremes of container hosting in Azure:
1. **Azure Container Instances (ACI):** Extremely simple, but lacks ingress routing, traffic splitting, microservice discovery, and auto-scaling.
2. **Azure Kubernetes Service (AKS):** Extremely powerful, but requires deep Kubernetes expertise, complex YAML, helm charts, and dedicated node pool costs.

**Azure Container Apps (ACA)** fills the sweet spot:
> **"All the power of Kubernetes and microservices, with zero cluster management overhead."**

```
+--------------------------------------------------------------------------+
|                  THE AZURE CONTAINER SPECTRUM                            |
+--------------------------------------------------------------------------+
|  [ Azure ACI ]              [ Azure Container Apps ]      [ Azure AKS ]  |
|  - Single Container         - Microservices               - Enterprise K8s|
|  - No Auto-scaling          - Scale-to-Zero               - Custom Helm   |
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
|   |   - Automatic HTTPS Termination                                  |   |
|   |   - Traffic Splitting (e.g., 80% to v1, 20% to v2 Canary)        |   |
|   +---------------------------------+--------------------------------+   |
|                                     |                                    |
|                                     v                                    |
|   +------------------------------------------------------------------+   |
|   |   APPLICATION CONTAINERS (Your Microservice Code)                |   |
|   |   - Revisions & Blue/Green Deployments                           |   |
|   +---------------------------------+--------------------------------+   |
|                                     |                                    |
|                                     v                                    |
|   +------------------------------------------------------------------+   |
|   |   AUTOSCALING ENGINE (KEDA - Kubernetes Event-Driven Autoscaling)|   |
|   |   - Scale based on HTTP requests, CPU, RAM, or Azure Queue messages  |
|   |   - SCALE TO ZERO (Idle apps consume $0!)                        |   |
|   +------------------------------------------------------------------+   |
|                                                                          |
+--------------------------------------------------------------------------+
```

### Key Capabilities:
- **Scale-to-Zero:** If nobody visits your website or microservice at 3:00 AM, the number of running instances drops to `0`. You are billed **$0.00**. When a request arrives, ACA rapidly spins up a container in seconds!
- **Traffic Splitting (Canary Releases):** Easily route 90% of user traffic to your stable `v1` revision and 10% to your new `v2` revision to test new features safely.
- **Revisions Management:** Every code update creates an immutable revision, allowing one-click rollbacks if a bug is introduced.

---

## 3. Comparing ACI, ACA, and AKS

| Feature | ACI | ACA | AKS |
| :--- | :--- | :--- | :--- |
| **Primary Use Case** | Batch tasks, isolated runs | Web apps, APIs, microservices | Complex enterprise orchestrations |
| **Scale to Zero** | No (You stop the container) | **Yes (Automatic when idle)** | Complex (requires virtual nodes/KEDA) |
| **Built-in HTTPS Ingress**| No | **Yes (Free automatic SSL)** | Requires Ingress Controller (Nginx/Traefik) |
| **Traffic Splitting** | No | **Yes (Slider-based % routing)**| Requires Service Mesh (Istio/Linkerd) |
| **Kubernetes Knowledge Needed** | None | **None** | High |

---

## 4. Hands-on Lab: Deploying a Container App in the Portal

### Step 1: Create the Container App
1. Open the [Azure Portal](https://portal.azure.com).
2. Search for **Container Apps** and click **+ Create**.
3. **Basics Tab:**
   - **Subscription:** Select your subscription.
   - **Resource group:** `rg-aca-demo`.
   - **Container App name:** `my-microservice-app`.
   - **Region:** `East US`.
   - **Container Apps Environment:** Click *Create new*:
     - Environment name: `cae-production-environment`.
     - Zone redundancy: *Disabled* (for dev/test).
     - Click **Create**.

---

### Step 2: Configure the Container Image
1. Go to the **App settings** (or **Container**) tab.
2. Uncheck *Use quickstart image* if you want custom, or leave checked to practice with Microsoft's sample image:
   - **Image:** `mcr.microsoft.com/k8se/quickstart:latest`
   - **CPU:** `0.25 vCPU` (Ultra-low cost)
   - **Memory:** `0.5 GiB`

---

### Step 3: Configure Ingress (Networking)
1. Go to the **Ingress** tab:
   - **Ingress:** `Enabled`
   - **Ingress traffic:** `Accepting traffic from anywhere` (Public internet)
   - **Insecure connections:** Leave unchecked (Forces HTTPS)
   - **Target port:** `80` (The port the container listens on)
2. Click **Review + create** -> **Create**.

---

### Step 4: Verify Live Application & Revisions
1. Once deployed, open your Container App **Overview** page.
2. Copy the **Application Url** (e.g., `https://my-microservice-app.victoriouscliff-12345.eastus.azurecontainerapps.io`).
3. Open the link in a browser. You will see the live application served over secure HTTPS!

---

### Step 5: Traffic Splitting (Canary Deployment Demonstration)
1. In the left menu under **Application**, click **Revisions and replicas**.
2. Deploy a new revision with an updated container image.
3. Once both revisions are listed:
   - Click **Traffic splitting**.
   - Assign `80%` traffic to `Revision 1`.
   - Assign `20%` traffic to `Revision 2`.
   - Click **Save**.
4. The Envoy proxy immediately splits user requests without any server restarts or downtime!

---

## 5. Summary Checklist
- [x] ACA is a serverless container platform tailored for microservices and APIs.
- [x] Powered by Kubernetes, KEDA, and Envoy without exposing K8s management complexity.
- [x] Supports automatic scale-to-zero to minimize cloud hosting costs.
- [x] Includes out-of-the-box HTTPS ingress and percentage-based traffic splitting for canary deployments.
