# 08 - Serverless Containers: Azure Container Instances (ACI)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:23:32` to `04:29:29`

---

## 1. What is Azure Container Instances (ACI)?

**Azure Container Instances (ACI)** is the fastest and simplest way to run a Docker container in Microsoft Azure.

In the previous module, we saw that running containers in Kubernetes (AKS) requires provisioning node pools, writing complex YAML manifests, configuring ingress controllers, and paying for underlying worker VMs even when no apps are handling traffic.

ACI is completely **Serverless**:
- **Zero VM Management:** You never see, configure, or patch an operating system.
- **Zero Cluster Overhead:** No Kubernetes master nodes, kubeconfig files, or cluster upgrades.
- **Pay-Per-Second Billing:** You pay strictly for the exact vCPU and memory allocated while the container is executing. When the container stops, billing halts instantly!
- **Fast Startup:** Container execution begins in 5 to 10 seconds.

```
+-----------------------------------------------------------------------+
|                    WHEN TO CHOOSE ACI VS. AKS                         |
+-----------------------------------------------------------------------+
|                                                                       |
|   Need to run a quick test container?                                 |
|   Executing a scheduled nightly batch job / ETL? ===> [ Azure ACI ]   |
|   Running a short-lived image processing task?         (Serverless)   |
|                                                                       |
|   Running 50 interconnected microservices?                            |
|   Need service mesh, rolling updates, auto-healing? ===> [ Azure AKS ]|
|   Enterprise production web platforms?                 (Kubernetes)   |
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
| **Best For** | Monolithic legacy applications | Complex microservice architectures | Short-lived tasks, CI/CD runners, quick demos |

---

## 3. Understanding ACI Restart Policies

When launching a container in ACI, you configure a **Restart Policy**:

| Restart Policy | When it restarts | Best Used For |
| :--- | :--- | :--- |
| **Always** (Default) | Automatically restarts if the container stops or crashes. | Long-running web applications and public APIs that must stay online 24/7. |
| **Never** | Runs once and stops permanently regardless of exit code. | One-time migration tasks or single-execution automation scripts. |
| **OnFailure** | Restarts only if the container exits with an error (non-zero exit code). | Scheduled ETL data processing and batch compute jobs. |

---

## 4. Hands-on Lab: Deploying a Container to ACI in the Portal

In this lab from the video, M Prashant demonstrates launching a public-facing web server in under 60 seconds without managing any infrastructure:

### Step 1: Create Container Instance in Azure Portal
1. Open the [Azure Portal](https://portal.azure.com).
2. Search for **Container instances** and click **+ Create**.

#### Tab 1: Basics
- **Subscription:** Select your subscription.
- **Resource group:** `rg-aci-demo`.
- **Container name:** `aci-web-quickstart`.
- **Region:** `East US`.
- **Image source:** **Quickstart images** (or *Docker Hub / Azure Container Registry*).
- **Image:** `mcr.microsoft.com/azuredocs/aci-helloworld:latest` (Linux).
- **OS type:** `Linux`.
- **Size:** `1 vCPU`, `1.5 GiB memory`.

---

#### Tab 2: Networking & DNS
- **Networking type:** **Public**.
- **DNS name label:** Enter a unique name, e.g.:
  `myaciwebdemo-prashant`
  *(Azure generates the FQDN: `myaciwebdemo-prashant.eastus.azurecontainer.io`)*.
- **Ports:**
  - Port: `80`
  - Protocol: `TCP`

---

#### Tab 3: Advanced
- **Restart policy:** **Always** (Keeps our web server online).
- **Environment variables:** (Optional - add key-value pairs like `ENV=Production`).

---

#### Tab 4: Review + Create
1. Click **Review + create** -> **Create**.
2. Within **15 to 30 seconds**, the container status turns to **Running**!

---

### Step 2: Verify the Live Container
1. On the Container Instance **Overview** page, locate the **FQDN** (Fully Qualified Domain Name).
2. Open your web browser and navigate to:
   `http://myaciwebdemo-prashant.eastus.azurecontainer.io`
3. The page displays: **"Welcome to Azure Container Instances!"**

---

## 5. Deploying and Managing ACI via Azure CLI

You can launch, inspect, and delete container instances with single CLI commands:

```bash
# 1. Create a Resource Group
az group create --name rg-aci-demo --location eastus

# 2. Deploy a public web container with custom DNS in seconds
az container create \
  --resource-group rg-aci-demo \
  --name aci-web-quickstart \
  --image mcr.microsoft.com/azuredocs/aci-helloworld:latest \
  --dns-name-label myaciwebdemo-$RANDOM \
  --ports 80 \
  --cpu 1 \
  --memory 1.5 \
  --restart-policy Always

# 3. Stream real-time container logs (like 'docker logs -f')
az container logs --resource-group rg-aci-demo --name aci-web-quickstart --follow

# 4. Attach an interactive terminal inside the container (like 'docker exec -it')
az container exec --resource-group rg-aci-demo --name aci-web-quickstart --exec-command "/bin/sh"

# 5. Stop the container (Billing stops immediately!)
az container stop --resource-group rg-aci-demo --name aci-web-quickstart

# 6. Delete the container instance
az container delete --resource-group rg-aci-demo --name aci-web-quickstart --yes
```

---

## 6. Mounting Persistent Storage (Azure Files to ACI)

Containers are ephemeral: when a container stops, all files written inside its local filesystem are destroyed. To persist data (e.g., application logs or uploaded user images), you can mount an **Azure File Share** directly into an ACI container:

```bash
# Mount an Azure File Share as a persistent volume inside /data
az container create \
  --resource-group rg-aci-demo \
  --name aci-with-storage \
  --image nginx \
  --azure-file-volume-account-name mystoragedemo123 \
  --azure-file-volume-account-key "YOUR_STORAGE_KEY" \
  --azure-file-volume-share-name corporate-share \
  --azure-file-volume-mount-path /usr/share/nginx/html
```

---

## 7. Troubleshooting Common ACI Issues

| Problem | Cause | Resolution |
| :--- | :--- | :--- |
| DNS Name label creation fails with `AlreadyExists`. | The DNS prefix must be globally unique across all of Azure. | Append a random number or your name (e.g., `myweb-prashant-8921`). |
| Container status shows `Waiting` or `Terminated`. | Private Docker Hub or ACR image cannot be pulled. | Provide image registry login credentials via `--registry-login-server`, `--registry-username`, and `--registry-password`. |
| Port 80 not reachable in browser. | Target container does not listen on port 80. | Verify internal application port matches the `--ports` argument. |

---

## 8. Summary Checklist
- [x] ACI is a serverless container engine with zero VM or cluster overhead.
- [x] Containers launch in seconds with per-second compute billing.
- [x] Ideal for rapid testing, scheduled batch ETL, and short-lived CI/CD jobs.
- [x] Supports persistent data storage via Azure Files volume mounting.
- [x] Generates instant public endpoints with custom DNS labels.
