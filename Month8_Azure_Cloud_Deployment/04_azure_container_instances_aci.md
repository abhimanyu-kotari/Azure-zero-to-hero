# 🚀 04 - Serverless Containers: Azure Container Instances (ACI)

<div align="center">

![Microsoft Azure](https://img.shields.io/badge/Azure_Container_Instances-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Serverless](https://img.shields.io/badge/Compute-Serverless-green?style=for-the-badge)
![Billing](https://img.shields.io/badge/Billing-Per--Second-yellow?style=for-the-badge)
![Fast Deploy](https://img.shields.io/badge/Startup-5--10s-blue?style=for-the-badge)

</div>

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:23:32` to `04:29:29` (Pure Serverless Containers)

---

## ⚡ 1. Why ACI? The Problem with VMs & Kubernetes

In the previous modules, we saw two ways to run software in the cloud:
1. **Azure Virtual Machines (IaaS):** You must provision a full operating system, manage security patches, configure firewall ports, and pay for the VM 24/7 even when nobody is using it.
2. **Azure Kubernetes Service (AKS):** Extremely powerful, but requires configuring node pools, writing complex YAML deployment manifests, and paying for underlying worker VMs continuously.

What if you just want to run **a single Docker container** for 10 minutes to process images, run a database migration script, or host a quick test website? You do NOT want to spend 20 minutes setting up a Kubernetes cluster!

**Azure Container Instances (ACI) is the solution:**
> ACI is a **pure Serverless container execution engine**. You supply the container image name, and Azure runs it in seconds without you ever seeing or managing a single virtual machine!

```
+---------------------------------------------------------------------------------------------------+
|                                     AZURE COMPUTE SPECTRUM                                        |
+---------------------------------------------------------------------------------------------------+
|  [ Azure Virtual Machine ]     --->  [ Azure Kubernetes (AKS) ]    --->  [ Azure Container Instances ]|
|  - Full OS Control                   - Complex Cluster Orchestration      - PURE SERVERLESS CONTAINER     |
|  - You patch the OS                  - Worker Node VMs                    - Zero VM Management            |
|  - Takes minutes to boot             - Best for 50+ microservices         - Starts in 5 to 10 seconds!    |
|  - Billed 24/7                       - Billed 24/7 for Node Pools         - Billed per-second while active|
+---------------------------------------------------------------------------------------------------+
```

---

## 2. Comparing Azure Compute for Containers

| Architectural Feature | Azure Virtual Machines (VM) | Azure Kubernetes Service (AKS) | Azure Container Instances (ACI) |
| :--- | :--- | :--- | :--- |
| **Abstraction Level** | Raw Infrastructure (IaaS) | Container Orchestrator (PaaS/Managed) | **Pure Serverless Container** |
| **Startup Speed** | 1 to 3 minutes | 1 to 2 minutes | **5 to 15 seconds** |
| **Underlying Host OS** | Fully exposed; patched by you | Managed worker VMs in node pools | **Completely hidden & abstracted** |
| **Billing Model** | Billed continuously per hour | Billed continuously for worker VMs | **Pay strictly per-second while running** |
| **Cluster Management**| Manual scaling or VMSS | Kubernetes API, Helm, Ingress | **Zero cluster setup required** |
| **Primary Use Cases** | Monolithic legacy systems | Complex enterprise microservices | **Batch jobs, scripts, ETL, quick APIs** |

---

## 3. Understanding ACI Restart Policies

When you launch a container in ACI, you must choose how Azure should handle container completion or crashes:

```
+-----------------------------------------------------------------------+
|                         ACI RESTART POLICIES                          |
+-----------------------------------------------------------------------+
|                                                                       |
|  1. Always (Default):                                                 |
|     If container stops or crashes, Azure restarts it immediately.     |
|     Best for: Web applications, APIs, background listener daemons.   |
|                                                                       |
|  2. Never:                                                            |
|     Container executes once to completion and STOPS permanently.      |
|     Best for: One-off database migrations, single calculation jobs.   |
|                                                                       |
|  3. OnFailure:                                                        |
|     Restarts ONLY if the process exits with an error code (non-zero). |
|     Best for: Nightly automated backup scripts, scheduled ETL jobs.   |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 4. Hands-on Lab: Deploying a Web Container to ACI in the Portal

In this lab from the video, M Prashant demonstrates launching a public-facing web server in under 60 seconds with an automatic public DNS address:

### Step 1: Create Container Instance in Azure Portal
1. Open the [Azure Portal](https://portal.azure.com).
2. In the top search bar, search for **Container instances** and click **+ Create**.

#### Tab 1: Basics
- **Subscription:** Select your subscription.
- **Resource group:** Select existing or click *Create new* $\rightarrow$ `rg-aci-demo`.
- **Container name:** `aci-web-quickstart`.
- **Region:** `East US` (or nearest region).
- **Image source:** Select **Quickstart images** (or *Docker Hub / Azure Container Registry*).
- **Image:** Select `mcr.microsoft.com/azuredocs/aci-helloworld:latest` (Linux).
- **OS type:** `Linux`.
- **Size:** Click *Change size* $\rightarrow$ `1 vCPU`, `1.5 GiB memory` (Cheapest tier).

---

#### Tab 2: Networking & Public DNS
- **Networking type:** Select **Public**.
- **DNS name label:** Enter a globally unique prefix, e.g.:
  `myaciwebdemo-prashant`
  *(Azure automatically constructs the full public FQDN: `myaciwebdemo-prashant.eastus.azurecontainer.io`)*.
- **Ports:**
  - Port: `80`
  - Protocol: `TCP`

---

#### Tab 3: Advanced
- **Restart policy:** Select **Always** (Keeps our web server running continuously).
- **Environment variables:** (Optional - add key-value pairs like `ENVIRONMENT=Production`).

---

#### Tab 4: Review + Create
1. Click **Review + create** $\rightarrow$ click **Create**.
2. Within **15 to 30 seconds**, deployment completes and status displays **Running**!

---

### Step 2: Verify the Live Container in Your Browser
1. Click **Go to resource** to open the Container Instance overview page.
2. In the top right metadata section, copy the **FQDN** (e.g., `myaciwebdemo-prashant.eastus.azurecontainer.io`).
3. Open a browser tab and navigate to:
   `http://myaciwebdemo-prashant.eastus.azurecontainer.io`
4. **Result:** The official **"Welcome to Azure Container Instances!"** landing page loads instantly!
5. Notice: You deployed a public web application without configuring a single VM, NIC, or web server daemon!

---

## 5. Managing ACI via Azure CLI Command Suite

You can automate ACI completely via the Azure CLI:

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

# 3. Stream real-time container logs to your terminal (like 'docker logs -f')
az container logs --resource-group rg-aci-demo --name aci-web-quickstart --follow

# 4. Attach an interactive shell inside the container (like 'docker exec -it')
az container exec --resource-group rg-aci-demo --name aci-web-quickstart --exec-command "/bin/sh"

# 5. Stop the container (Compute billing ceases immediately!)
az container stop --resource-group rg-aci-demo --name aci-web-quickstart

# 6. Restart the container
az container start --resource-group rg-aci-demo --name aci-web-quickstart

# 7. Delete the container instance
az container delete --resource-group rg-aci-demo --name aci-web-quickstart --yes
```

---

## 6. Attaching Persistent Storage: Azure Files to ACI

Containers are inherently stateless and ephemeral: when a container terminates, any files created inside its local filesystem are permanently wiped.

To persist data (e.g. database data files or uploaded user documents), ACI can mount an **Azure File Share** directly into a folder inside the container:

```bash
# Mount an Azure File Share into /usr/share/nginx/html inside the container
az container create \
  --resource-group rg-aci-demo \
  --name aci-with-persistent-storage \
  --image nginx \
  --azure-file-volume-account-name mystorageacct123 \
  --azure-file-volume-account-key "YOUR_STORAGE_ACCOUNT_KEY" \
  --azure-file-volume-share-name corporate-share \
  --azure-file-volume-mount-path /usr/share/nginx/html \
  --dns-name-label mynginx-persisted-$RANDOM \
  --ports 80
```

---

## 7. Troubleshooting Common ACI Issues

| Problem | Root Cause | Resolution |
| :--- | :--- | :--- |
| Creation fails with `DnsNameLabelUnavailable`. | The DNS prefix is already taken by another Azure customer globally. | Change the DNS name label by appending random digits (e.g., `myapp-student-9812`). |
| Container status is stuck in `Waiting` or `Terminated`. | Container image cannot be pulled (private Docker Hub / ACR credentials missing). | Provide image registry credentials using `--registry-login-server`, `--registry-username`, and `--registry-password`. |
| Browser shows `Connection Refused` on port 80. | The containerized app is listening on a different port (e.g., 3000, 8080). | Match the `--ports` argument to the exact internal port the containerized process exposes. |

---

## 8. Summary Checklist
- [x] ACI is Azure's fastest serverless container deployment mechanism (starts in seconds).
- [x] Billed per-second strictly while the container is executing.
- [x] Automatically provides public IPs and custom FQDN domain labels.
- [x] Supports three restart policies: Always (Web apps), Never (Scripts), and OnFailure (ETL).
- [x] Can mount persistent Azure File Shares to prevent data loss.
