# 08 - Serverless Containers: Azure Container Instances (ACI)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:23:32` to `04:29:29`

---

## 1. What is Azure Container Instances (ACI)?

**Azure Container Instances (ACI)** is the fastest and simplest way to run a Docker container in Microsoft Azure.

In the previous module, we learned that running containers in Kubernetes (AKS) requires provisioning node pools, configuring YAML manifests, managing ingress controllers, and paying for underlying VMs even when no apps are processing traffic.

ACI is completely **Serverless**:
- **Zero VM Management:** You never see, configure, or patch an operating system.
- **Zero Cluster Management:** No Kubernetes master nodes, kubeconfig files, or cluster updates.
- **Pay-Per-Second Billing:** You pay strictly for the exact vCPU and memory allocated while the container is executing. When the container stops, billing stops immediately.
- **Lightning Fast Startup:** Launches containers in seconds.

```
+-----------------------------------------------------------------------+
|                    WHEN TO CHOOSE ACI VS. AKS                         |
+-----------------------------------------------------------------------+
|                                                                       |
|   Need a quick test container?                                        |
|   Running a scheduled batch job / nightly ETL?   ===> [ Azure ACI ]   |
|   Processing a single video or file conversion?       (Serverless)    |
|                                                                       |
|   Running 50 interconnected microservices?                            |
|   Need service mesh, rolling updates, auto-healing? ===> [ Azure AKS ]|
|   High-scale production enterprise web apps?         (Kubernetes)     |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 2. Comparing Azure Compute for Containers

| Feature | Azure Virtual Machines | Azure Kubernetes Service (AKS) | Azure Container Instances (ACI) |
| :--- | :--- | :--- | :--- |
| **Abstraction Level** | Raw Infrastructure (IaaS) | Container Orchestration (PaaS/Managed) | Pure Serverless Container |
| **Startup Speed** | 1 to 3 minutes | 1 to 2 minutes | **5 to 15 seconds** |
| **Underlying Host** | Visible, patched by you | Visible VMs in your node pool | **Completely hidden & abstracted** |
| **Cost Model** | Billed 24/7 for VM size | Billed 24/7 for worker node VMs | **Per-second billing while running** |
| **Scaling** | Manual or VMSS | Horizontal Pod Autoscaler (HPA) | Manual or event-triggered |
| **Best For** | Legacy monolithic software | Complex microservice architectures | Short-lived tasks, CI/CD runners, quick demos |

---

## 3. Hands-on Lab: Deploying a Container to ACI in the Portal

In this lab from the video, M Prashant demonstrates launching a public-facing web server in under 60 seconds without managing any infrastructure:

### Step 1: Create Container Instance
1. Open the [Azure Portal](https://portal.azure.com).
2. Search for **Container instances** and click **+ Create**.
3. **Basics Tab:**
   - **Subscription:** Select your subscription.
   - **Resource group:** `rg-aci-demo`.
   - **Container name:** `aci-web-quickstart`.
   - **Region:** `East US`.
   - **Image source:** **Quickstart images** (or *Docker Hub / Other registry*).
   - **Image:** `mcr.microsoft.com/azuredocs/aci-helloworld:latest` (Linux).
   - **Size:** 1 vCPU, 1.5 GiB memory.

---

### Step 2: Configure Networking & DNS
1. Navigate to the **Networking** tab.
2. **Networking type:** **Public**.
3. **DNS name label:** Enter a unique name, e.g.:
   `myaciwebdemo-prashant`
   *(Azure will give you the FQDN: `myaciwebdemo-prashant.eastus.azurecontainer.io`)*
4. **Ports:**
   - Port: `80`
   - Protocol: `TCP`

---

### Step 3: Review + Create
1. Click **Review + create** -> **Create**.
2. Within **15 to 30 seconds**, the container status turns to **Running**!

---

### Step 4: Verify the Container
1. On the Container Instance **Overview** page, locate the **FQDN** (Fully Qualified Domain Name).
2. Open your web browser and navigate to:
   `http://myaciwebdemo-prashant.eastus.azurecontainer.io`
3. The page displays: **"Welcome to Azure Container Instances!"**

---

## 4. Deploying via Azure CLI

You can launch the exact same container with a single command from your terminal:

```bash
# 1. Create a resource group
az group create --name rg-aci-demo --location eastus

# 2. Deploy the container instance with public DNS
az container create \
  --resource-group rg-aci-demo \
  --name aci-web-quickstart \
  --image mcr.microsoft.com/azuredocs/aci-helloworld:latest \
  --dns-name-label myaciwebdemo-$RANDOM \
  --ports 80 \
  --cpu 1 \
  --memory 1.5

# 3. View container logs (like running 'docker logs')
az container logs --resource-group rg-aci-demo --name aci-web-quickstart

# 4. Delete the container instance when finished
az container delete --resource-group rg-aci-demo --name aci-web-quickstart --yes
```

---

## 5. Summary Checklist
- [x] ACI is a serverless container engine with zero VM or cluster overhead.
- [x] Ideal for rapid testing, scheduled data pipelines, batch image processing, and CI/CD jobs.
- [x] Containers spin up in seconds and charge strictly per second of CPU and memory used.
- [x] Generates instant public endpoints with custom DNS labels.
