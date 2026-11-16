# 04 - Azure Container Instances (ACI)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:23:32` to `04:29:29`

---

## 1. What is Azure Container Instances (ACI)?

Imagine you have created a Docker container image for your application, and you want to run it on Azure right now. 

Normally, you would have to:
1. Create a Virtual Machine.
2. Install Docker on the VM.
3. Configure networking and firewalls.
4. Keep the VM running 24/7 (and pay for it even when no one is using your app).

**Azure Container Instances (ACI)** eliminates all of this hassle!

**ACI is the simplest, fastest, and most straightforward way to run a Docker container in the cloud without managing any virtual machines, and without having to learn complex tools like Kubernetes.**

It is a **Serverless Container** service: you simply point Azure to your Docker image, click run, and within seconds your container is live on the internet with its own public IP address.

---

## 2. Key Features of ACI

### 1. Blazing Fast Startup (Seconds, Not Minutes)
Because ACI does not boot up a full operating system or provision a VM, your container starts up in just a few seconds.

### 2. Exact Resource Sizing
You can request exactly the amount of CPU and memory your application needs:
- For example: `1.0 vCPU` and `1.5 GiB Memory`.
- You don't have to overpay for a large VM size if your app only needs a small amount of compute.

### 3. Per-Second Billing (Save Money)
With ACI, you are billed strictly for the exact seconds your container is running:
- If a data-processing script runs for 3 minutes and finishes, you pay only for those 3 minutes.
- When the container stops, your billing drops immediately to zero!

### 4. Built-in Public IP & Custom DNS Name
Azure provides a public IP and an automatic domain name (e.g., `my-student-app.eastus.azurecontainer.io`) so anyone in the world can access your web container over the internet immediately.

### 5. Persistent Storage Mounting
Containers are naturally ephemeral (temporary), meaning any file created inside disappears when the container restarts. ACI allows you to mount **Azure File Shares** directly as a folder inside the container so your data is permanently preserved.

---

## 3. Container Restart Policies

When launching an ACI container, you choose a **Restart Policy** based on what your code does:

1. **`Always` (Default for Web Apps):**
   - If the container stops or crashes, Azure automatically restarts it immediately.
   - **Use Case:** Web applications, APIs, or websites that must stay online 24/7.
2. **`OnFailure`:**
   - Azure restarts the container only if it exits with an error (non-zero exit code). If it completes successfully, it stays stopped.
   - **Use Case:** Background tasks that need a second chance if a network glitch occurs.
3. **`Never`:**
   - Runs once and shuts down forever, regardless of success or failure.
   - **Use Case:** One-time database schema migrations, daily reporting scripts, or batch calculations.

---

## 4. Hands-on Walkthrough: Deploying an ACI Container in Azure Portal

In the video, M Prashant demonstrates deploying a sample web container in under two minutes:

### Step 1: Create Container Instance
1. Open the [Azure Portal](https://portal.azure.com) and search for **Container instances**.
2. Click **Create**.
3. **Basics:**
   - **Subscription:** Select your subscription.
   - **Resource group:** `rg-containers-demo`.
   - **Container name:** `student-web-aci`.
   - **Region:** e.g., *East US*.
   - **Image source:** Select **Quickstart images**.
   - **Image:** Select `mcr.microsoft.com/azuredocs/aci-helloworld:latest` (Linux).
   - **Size:** Leave default (1 vCPU, 1.5 GiB memory).

### Step 2: Networking Configuration
1. Click the **Networking** tab.
2. **Networking type:** Select **Public**.
3. **DNS name label:** Enter a unique name (e.g., `student-aci-test-123`).
4. **Ports:** Ensure port `80` (TCP) is open.

### Step 3: Review and Deploy
1. Click **Review + Create**, then click **Create**.
2. Wait approximately 20 to 30 seconds for the deployment to finish.
3. Click **Go to resource**.
4. Copy the **FQDN (Fully Qualified Domain Name)**:
   ```
   http://student-aci-test-123.eastus.azurecontainer.io
   ```
5. Paste it into your browser. You will see the **"Welcome to Azure Container Instances!"** live webpage running on Microsoft's cloud!

---

## 5. When to Use ACI (Best Use Cases)

- **Scheduled & Batch Jobs:** A script that wakes up every midnight, processes invoices, and shuts down after 10 minutes.
- **CI/CD Build Runners:** Running test suites or compiling code inside an isolated container during GitHub or Azure DevOps pipelines.
- **Data Transformation & ETL:** Converting, resizing, or analyzing video/image files upon upload.
- **Quick Prototyping:** Testing a Docker container you just built without having to set up a VM or cluster.

---

## 6. Limitations of ACI: Why Isn't It Always Enough?

While ACI is great for single, standalone containers, it has limitations when building large modern applications:
- **No Autoscaling:** ACI cannot automatically scale out (e.g. from 1 container to 10 containers) when web traffic spikes.
- **No Ingress Traffic Routing:** Cannot split traffic (e.g., 90% traffic to Version 1, 10% traffic to Version 2).
- **No Microservices Features:** Does not have built-in service discovery or communication tools between multiple services.

> **Next Step:** To solve these exact limitations, Microsoft created **Azure Container Apps (ACA)**!

---

## 7. Summary Checklist
- [x] ACI is the quickest serverless way to run a Docker container in Azure.
- [x] Zero virtual machine setup or operating system maintenance required.
- [x] Billed per second of execution; perfect for short-lived tasks and batch jobs.
- [x] Supports public IP addresses, custom DNS names, and persistent Azure File Share mounts.
