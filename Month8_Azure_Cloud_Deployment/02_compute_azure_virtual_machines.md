# 02 - Compute: Azure Virtual Machines (VMs)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `00:51:43` to `01:44:39`

---

## 1. What is Azure Compute?

In cloud computing, **Compute** refers to the processing power, memory (RAM), and CPU cycles needed to run your software, execute scripts, and host websites. 

Microsoft Azure offers several compute options depending on your needs:
- **Virtual Machines (IaaS):** Complete control over the operating system.
- **Containers (ACI / ACA / AKS):** Lightweight, packaged applications.
- **Serverless (Azure Functions):** Code executed purely in response to triggers.

The most fundamental compute service in Azure is the **Azure Virtual Machine (VM)**.

---

## 2. What is an Azure Virtual Machine?

An **Azure Virtual Machine** is an on-demand, scalable computing resource that behaves just like a physical laptop or desktop computer, but lives in Microsoft's secure datacenter. 

When you create a VM, you choose:
- The **Operating System (OS):** Ubuntu Linux, Debian, Red Hat Enterprise Linux, Windows Server, or Windows 11.
- The **CPU cores and RAM:** From 1 core and 1 GB RAM up to hundreds of cores and terabytes of memory.
- The **Storage Disks:** Hard drives attached to store your operating system and files.
- The **Network:** Private IP address and optional Public IP address to connect from the internet.

---

## 3. Understanding VM Sizing & Families

Azure names its virtual machine sizes using letters. Each "family" is designed for a specific type of workload:

| VM Series | Category | Best Used For |
| :--- | :--- | :--- |
| **B-Series** | Burstable (Economical) | Development, testing, low-traffic websites, student learning. You get baseline performance and save credits when idle. |
| **D-Series** | General Purpose | Balanced CPU-to-memory ratio. Ideal for production web servers, medium databases, and enterprise apps. |
| **E-Series** | Memory Optimized | High memory-to-CPU ratio. Best for large relational databases (SQL Server), in-memory caches (Redis), and data analytics. |
| **F-Series** | Compute Optimized | High CPU-to-memory ratio. Best for batch processing, video encoding, and gaming servers. |
| **N-Series** | GPU Enabled | Equipped with NVIDIA GPUs for artificial intelligence (AI), machine learning training, and 3D rendering. |

> **Student Tip:** For practice and labs, always select the **B1s** or **B2s** size (eligible for Azure Free Tier) to avoid burning your free trial credits!

---

## 4. Step-by-Step: Creating a Linux VM in the Azure Portal

In the video demonstration, M Prashant guides us through launching an Ubuntu Linux VM from scratch:

### Step 1: Basics Configuration
1. Open the [Azure Portal](https://portal.azure.com) and search for **Virtual Machines** in the search bar.
2. Click **Create** -> **Azure virtual machine**.
3. **Project Details:**
   - **Subscription:** Select your active subscription (e.g., *Azure subscription 1*).
   - **Resource Group:** Click *Create new* and name it `rg-compute-demo`.
4. **Instance Details:**
   - **Virtual machine name:** `my-first-linux-vm`.
   - **Region:** Choose a region close to you (e.g., *East US* or *Central India*).
   - **Availability Options:** Select *No infrastructure redundancy required* (for learning).
   - **Security type:** *Standard*.
   - **Image:** Select *Ubuntu Server 22.04 LTS - x64 Gen2*.
   - **Size:** Select `Standard_B1s` (1 vCPU, 1 GiB memory) or `Standard_B2s`.

### Step 2: Administrator Account (Authentication)
Azure offers two ways to log into your Linux VM:
- **SSH Public Key (Recommended & Secure):** Generates a cryptographic key pair (a public key on Azure and a private `.pem` file on your laptop). No password can be guessed by hackers.
- **Password:** A traditional username and password.

Choose **SSH public key**:
- **Username:** `azureuser`
- **SSH public key source:** *Generate new key pair*
- **Key pair name:** `my-first-linux-vm_key`

### Step 3: Inbound Port Rules (Firewall / Network Security Group)
Under **Public inbound ports**, choose **Allow selected ports**:
- Check **SSH (22):** Allows you to securely connect to the Linux command line.
- Check **HTTP (80):** Allows users to visit your web server via a browser.

### Step 4: Disks & Networking
- **OS Disk Type:** Select **Standard SSD** (good balance of speed and cost).
- **Networking:** Azure will automatically generate a new Virtual Network (VNet), Subnet, and Public IP address for you.

### Step 5: Review + Create
Click **Review + create**. Azure validates your settings. Click **Create**. A popup will appear asking you to download your private key. **Download the `.pem` file** and save it safely on your computer.

---

## 5. Connecting to Your Virtual Machine

Once the deployment status shows *Your deployment is complete*, click **Go to resource**. Copy your VM's **Public IP address** (e.g., `20.120.45.67`).

### Connecting via Terminal or PowerShell (Mac, Linux, Windows 10/11):
1. Open your terminal or PowerShell.
2. Navigate to the folder where you saved the `.pem` key file:
   ```bash
   cd ~/Downloads
   ```
3. Set read-only permissions on the private key (required by SSH):
   ```bash
   chmod 400 my-first-linux-vm_key.pem
   ```
4. Connect to the virtual machine using the `ssh` command:
   ```bash
   ssh -i my-first-linux-vm_key.pem azureuser@<YOUR_VM_PUBLIC_IP>
   ```
5. Type `yes` when prompted to accept the host fingerprint. You are now logged into your cloud server!

---

## 6. Hands-on Lab: Deploying an Nginx Web Server

To prove our cloud server can serve real web traffic to users worldwide, we install the **Nginx** web server:

```bash
# 1. Update the Linux package index
sudo apt update

# 2. Install Nginx web server
sudo apt install nginx -y

# 3. Verify that Nginx service is running
sudo systemctl status nginx
```

### Testing the Website:
1. Open a new tab in your web browser.
2. Type `http://<YOUR_VM_PUBLIC_IP>` (make sure you use `http://`, not `https://`).
3. You will immediately see the **"Welcome to nginx!"** default webpage served directly from your Azure VM!

---

## 7. Troubleshooting: Network Security Groups (NSGs)

What if you type the Public IP in your browser and the page just spins and times out?

### The Cause: Blocked Ports
An **NSG (Network Security Group)** acts as a virtual firewall for your VM. By default, Azure blocks all incoming traffic from the internet except the ports you explicitly allow.
- **Port 22:** Used for SSH terminal access.
- **Port 80:** Used for unencrypted HTTP web traffic.
- **Port 443:** Used for secure HTTPS web traffic.

### How to Fix:
1. Go to your VM in the Azure Portal.
2. In the left menu under **Settings**, click **Networking**.
3. Under **Inbound port rules**, check if port `80` is listed.
4. If missing, click **Add inbound port rule**:
   - **Destination port ranges:** `80`
   - **Protocol:** `TCP`
   - **Action:** `Allow`
   - **Priority:** `300`
   - **Name:** `Allow-HTTP-80`
5. Click **Add**. Within 10 seconds, refresh your browser tab, and your web page will load!

---

## 8. Summary Checklist
- [x] Azure Virtual Machines are on-demand IaaS compute instances.
- [x] Select B-series (burstable) for affordable learning and development.
- [x] Authenticate using SSH key pairs instead of passwords for superior security.
- [x] Network Security Groups (NSGs) control what traffic can reach your VM via port numbers.
- [x] Always stop or delete VMs when finished practicing to save credits.
