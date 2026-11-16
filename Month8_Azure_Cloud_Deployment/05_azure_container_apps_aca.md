# 05 - Azure Container Apps (ACA)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:29:29` to `04:38:36`

---

## 1. What is Azure Container Apps (ACA)?

While **Azure Container Instances (ACI)** is fantastic for running a single container for a quick task, what happens when you want to build a real-world web application with microservices, traffic routing, and automatic scaling?

Traditionally, you had to learn **Kubernetes**—which involves steep learning curves, hundreds of YAML files, and managing cluster nodes.

**Azure Container Apps (ACA)** is the modern solution:
It is a **fully managed serverless container platform** built on top of enterprise-grade open-source technologies (Kubernetes, KEDA, Envoy, and Dapr), but Microsoft handles 100% of the underlying infrastructure.

You get all the powerful features of Kubernetes **without needing to manage or even know how to run a Kubernetes cluster!**

---

## 2. Superpowers of Azure Container Apps

### 1. Scale-to-Zero (True Cost Efficiency)
- If nobody is visiting your website or using your API (e.g., at 3:00 AM), ACA automatically scales the number of running container replicas down to **zero**.
- When instances are at zero, **you pay $0 for compute resources!**
- The moment a new user visits the site, ACA detects the incoming HTTP request and immediately spins up a container in seconds to answer the request.

### 2. Event-Driven Autoscaling (Powered by KEDA)
ACA can scale your application up or down automatically based on:
- **HTTP Request Volume:** Add more replicas when concurrent web requests cross a certain threshold (e.g., more than 10 requests per container).
- **CPU & Memory Usage:** Scale up when containers cross 70% CPU utilization.
- **Queue Message Count:** If there are 500 unprocessed messages in an Azure Service Bus or RabbitMQ queue, ACA can spin up 10 background worker containers to process them simultaneously.

### 3. Built-In Ingress & Free HTTPS
- You don't need to configure reverse proxies (like Nginx) or purchase SSL certificates.
- ACA provides an **Ingress controller** out-of-the-box with **free, automatic TLS/SSL certificates** providing secure `https://` URLs immediately upon deployment.
- You can configure:
  - **External Ingress:** Accessible to anyone on the public internet (for frontend websites).
  - **Internal Ingress:** Accessible only to other microservices inside your private container environment (for private backend APIs or databases).

### 4. Revisions & Traffic Splitting (Canary / Blue-Green Deployments)
Every time you push a new code version or update your container image, ACA creates an immutable **Revision** (a snapshot of that version).
- You can run multiple revisions at the same time.
- **Traffic Splitting:** You can route 90% of user traffic to Revision 1 (the stable version) and 10% to Revision 2 (the new update) to test for bugs.
- If Revision 2 has a bug, you can revert 100% of traffic back to Revision 1 in **one second** without any downtime!

### 5. Microservices & Dapr Integration
ACA natively supports **Dapr (Distributed Application Runtime)**. This makes it trivial for microservices to talk to each other securely, store state, manage distributed locks, and publish/subscribe to message brokers without writing complex networking code.

---

## 3. Hands-on Walkthrough: Deploying a Container App

In the video, M Prashant demonstrates how fast you can launch a modern containerized web app:

### Step 1: Create Container App in Azure Portal
1. Search for **Container Apps** in the Azure Portal search bar and click **Create**.
2. **Basics Tab:**
   - **Subscription:** Select your subscription.
   - **Resource group:** `rg-containers-demo`.
   - **Container App name:** `my-first-container-app`.
   - **Region:** e.g., *East US*.
   - **Container Apps Environment:** Click *Create new* (e.g., `env-production-eastus`). An environment acts as a secure boundary sharing the same virtual network and logging workspace among multiple container apps.

### Step 2: Container Details
1. Uncheck *Use quickstart image* if you have your own image, or select the built-in demo image (`mcr.microsoft.com/k8se/quickstart:latest`).
2. Set CPU and Memory allocation (e.g., `0.5 vCPU`, `1.0 GiB RAM`).

### Step 3: Ingress Configuration
1. Click the **Ingress** tab.
2. Check **Enable Ingress**.
3. **Ingress traffic:** Select **Accepting traffic from anywhere** (External).
4. **Target Port:** Enter `80` (the port your application listens on inside the container).

### Step 4: Review and Deploy
1. Click **Review + Create**, then click **Create**.
2. Once complete, click **Go to resource**.
3. Look for the **Application URL**:
   ```
   https://my-first-container-app.polardo-xyz.eastus.azurecontainerapps.io
   ```
4. Click the URL. Your application is live with full HTTPS encryption, health monitoring, and autoscaling!

---

## 4. The Ultimate Cheat Sheet: ACI vs. ACA vs. AKS

When should you use which container service in Azure?

```
                       CONTAINER COMPLEXITY & CONTROL
Simple                                                                 Advanced
-------------------------------------------------------------------------------
  [ Azure Container Instances ] ---> [ Azure Container Apps ] ---> [ Azure Kubernetes Service ]
             (ACI)                               (ACA)                             (AKS)
```

| Feature | Azure Container Instances (ACI) | Azure Container Apps (ACA) | Azure Kubernetes Service (AKS) |
| :--- | :--- | :--- | :--- |
| **Complexity** | Extremely simple (Zero infrastructure). | Moderate (Simple portal settings). | High (Requires Kubernetes mastery). |
| **Primary Focus** | Single container tasks & batch jobs. | Microservices & Web applications. | Full enterprise cluster orchestration. |
| **Autoscaling** | No (Manual resize only). | **Yes (Automatic + Scale-to-Zero)**. | Yes (HPA + Cluster Autoscaler). |
| **Ingress / HTTPS** | Basic Public IP / DNS. | **Built-in Envoy with Free SSL**. | Configure Ingress Controllers manually. |
| **Traffic Splitting** | Not supported. | **Yes (Revisions & % traffic)**. | Requires Service Mesh (Istio/Linkerd). |
| **Under the Hood** | Isolated serverless hypervisor. | Managed AKS + KEDA + Dapr. | Raw Kubernetes control plane + VMs. |
| **Best Choice When...** | You have a one-off script, daily backup, or CI runner. | You want to deploy modern microservices without managing Kubernetes. | You require deep control over Kubernetes APIs, CRDs, and custom networking. |

---

## 5. Summary Checklist
- [x] ACA is the ideal choice for modern microservices and web apps on Azure.
- [x] Automatically scales out when traffic increases, and scales down to zero when idle to save money.
- [x] Built-in Ingress handles HTTPS routing and canary traffic splitting effortlessly.
- [x] Eliminates the operational complexity of managing raw Kubernetes clusters.
