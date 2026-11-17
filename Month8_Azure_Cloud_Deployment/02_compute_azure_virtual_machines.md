# 02 - Compute: Azure Virtual Machines (VMs)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `00:51:43` to `01:44:39`

---

## 1. What is Azure Compute?

In cloud computing, **Compute** refers to the processing power, memory (RAM), and CPU cycles needed to run your software, execute scripts, and host websites. 

Microsoft Azure offers several compute options depending on how much control vs. automation you need:
- **Virtual Machines (IaaS):** Complete control over the operating system, kernel, and software stack.
- **Virtual Machine Scale Sets (VMSS):** Auto-scaling groups of identical VMs behind a load balancer.
- **Containers (ACI / ACA / AKS):** Lightweight, packaged applications that boot in seconds without managing full operating systems.
- **Serverless (Azure Functions):** Event-driven code execution where you pay strictly per millisecond of compute time.

The foundational compute building block of Microsoft Azure is the **Azure Virtual Machine (VM)**.

```
+-----------------------------------------------------------------------+
|                         AZURE COMPUTE SPECTRUM                        |
+-----------------------------------------------------------------------+
|  IaaS (More Control)  <-------------------->  PaaS / Serverless (Less)|
|                                                                       |
|  [Azure VM]  --->  [VMSS]  --->  [AKS]  --->  [ACA]  --->  [Functions]|
|   Full OS           Cluster       K8s          App-focused   Pure Code|
+-----------------------------------------------------------------------+
```

---

## 2. What is an Azure Virtual Machine?

An **Azure Virtual Machine** is an on-demand, scalable computing resource that behaves just like a physical computer or server, but resides in Microsoft's hyper-secure enterprise datacenters.

When you create a VM, Azure orchestrates several underlying resources:

```
+------------------------------------------------------------------+
|                   AZURE VIRTUAL MACHINE ARCHITECTURE             |
+------------------------------------------------------------------+
|                                                                  |
|   +----------------------------------------------------------+   |
|   |                  VIRTUAL MACHINE (OS & RAM)              |   |
|   |         Ubuntu 22.04 LTS / Windows Server 2022           |   |
|   +----------------------------+-----------------------------+   |
|                                |                                 |
|                                v                                 |
|   +----------------------------------------------------------+   |
|   |             OS DISK (Standard SSD / Premium SSD)         |   |
|   +----------------------------------------------------------+   |
|                                |                                 |
|                                v                                 |
|   +----------------------------------------------------------+   |
|   |              NETWORK INTERFACE CARD (NIC)                |   |
|   |  - Private IP: 10.0.0.4 (internal subnet communication)   |   |
|   |  - Attached to: Virtual Network (VNet) & Subnet          |   |
|   +----------------------------+-----------------------------+   |
|                                |                                 |
|            +-------------------+-------------------+             |
|            |                                       |             |
|            v                                       v             |
|   +------------------+                   +-------------------+   |
|   |    PUBLIC IP     |                   |   NETWORK SECURITY|   |
|   |  (e.g., 20.x.x.x)|                   |     GROUP (NSG)   |   |
|   |  Internet Access |                   | Port 22, 80, 443  |   |
|   +------------------+                   +-------------------+   |
+------------------------------------------------------------------+
```

---

## 3. Understanding Azure VM Sizing & Series

Azure organizes VM hardware configurations into distinct "families" or "series" optimized for specific workloads:

| Series | Type | CPU : RAM Ratio | Best Use Cases |
| :--- | :--- | :--- | :--- |
| **B-Series** | Burstable | Dynamic (1:1 to 1:4) | **Dev/test environments, student practice, low-traffic blogs**. Accumulates credits when idle and bursts during traffic spikes. |
| **D-Series** | General Purpose | Balanced (1:4) | Enterprise web apps, small-to-medium databases, production microservices. |
| **E-Series** | Memory Optimized | High Memory (1:8) | In-memory caches (Redis), relational databases (SQL Server, PostgreSQL), heavy analytics. |
| **F-Series** | Compute Optimized | High CPU (1:2) | Batch processing, analytics algorithms, video encoding, gaming backends. |
| **N-Series** | GPU Accelerated | Specialized | Deep learning training, AI model inference, CAD rendering, video editing. |

> [!TIP]
> **Student Cost Saver:** When practicing, always select `Standard_B1s` (1 vCPU, 1 GiB RAM) or `Standard_B2s` (2 vCPU, 4 GiB RAM). These are eligible under Azure's monthly free limits or use negligible pay-as-you-go credits.

---

## 4. Step-by-Step: Creating a Linux VM via Azure Portal

Follow this exact walkthrough to configure every tab and parameter in the Azure Portal.

### Step 1: Navigate to Virtual Machines
1. Log into the [Azure Portal](https://portal.azure.com).
2. In the top search bar, type `Virtual Machines` and select **Virtual Machines** under services.
3. Click **+ Create** -> **Azure virtual machine**.

---

### Step 2: Configure the "Basics" Tab

```
+---------------------------------------------------------------------------+
| BASICS TAB CONFIGURATION                                                  |
+---------------------------------------------------------------------------+
| Project Details:                                                          |
|   Subscription:        [ Your Active Subscription ]                       |
|   Resource Group:      [ Create new: rg-compute-demo ]                    |
|                                                                           |
| Instance Details:                                                         |
|   Virtual Machine Name:[ vm-web-demo ]                                    |
|   Region:              [ (US) East US / Central India ]                   |
|   Availability Options:[ No infrastructure redundancy required ]          |
|   Security Type:       [ Standard ]                                       |
|   Image:               [ Ubuntu Server 22.04 LTS - x64 Gen2 ]             |
|   VM Architecture:     [ x64 ]                                            |
|   Size:                [ Standard_B1s (1 vCPU, 1 GiB RAM) ]               |
|                                                                           |
| Administrator Account:                                                    |
|   Authentication Type: [ SSH public key ]                                 |
|   Username:            [ azureuser ]                                      |
|   SSH Public Key Source:[ Generate new key pair ]                         |
|   Key Pair Name:       [ vm-web-demo_key ]                                |
|                                                                           |
| Inbound Port Rules:                                                       |
|   Public Inbound Ports:[ Allow selected ports ]                           |
|   Select Inbound Ports:[x] SSH (22)  [x] HTTP (80)  [ ] HTTPS (443)       |
+---------------------------------------------------------------------------+
```

* **Authentication Type:** Always choose **SSH public key** instead of Password. SSH keys use 2048-bit or 4096-bit RSA cryptographic pairs that cannot be brute-forced.
* **Inbound Ports:** Checking **SSH (22)** allows remote terminal administration; checking **HTTP (80)** allows public web browsing.

---

### Step 3: Configure the "Disks" Tab
- **OS Disk Type:** Select **Standard SSD LRS** (Low cost, reliable performance for learning) instead of Premium SSD.
- **Delete with VM:** Ensure `[x] Delete with VM` is checked so the disk is automatically cleaned up when deleting the virtual machine.

---

### Step 4: Configure the "Networking" Tab
- **Virtual Network (VNet):** Azure will automatically suggest `rg-compute-demo-vnet`. Leave as default.
- **Subnet:** Default `10.0.0.0/24`.
- **Public IP:** Automatically assigned.
- **NIC Network Security Group:** Select **Basic** (the ports 22 and 80 selected earlier will be mapped).
- **Delete public IP and NIC when VM is deleted:** Check `[x]` to prevent orphaned orphan resources incurring ongoing storage costs.

---

### Step 5: Configure the "Advanced" Tab (Custom Data / Automation)

> [!IMPORTANT]
> In the video, M Prashant explains how to run automated startup scripts via **Custom Data** (Cloud-Init). This script runs automatically with root privileges the very first time the VM boots up!

Scroll down to the **Custom data** input box in the **Advanced** tab and paste the following bash script:

```bash
#!/bin/bash
sudo apt update -y
sudo apt install -y apache2
sudo systemctl start apache2
sudo systemctl enable apache2

echo "<html><h1>Welcome to Apache Web Server on Azure VM!</h1></html>" | sudo tee /var/www/html/index.html
```

#### What this script does line-by-line:
1. `#!/bin/bash`: Informs the Linux OS to execute the script using the Bash shell.
2. `sudo apt update -y`: Updates the package repository index so the system fetches the newest packages.
3. `sudo apt install -y apache2`: Installs the Apache HTTP web server non-interactively (`-y` confirms prompts).
4. `sudo systemctl start apache2`: Starts the Apache web service immediately.
5. `sudo systemctl enable apache2`: Configures Apache to automatically restart if the VM reboots.
6. `echo "<html><h1>...</h1></html>" | sudo tee /var/www/html/index.html`: Writes a custom webpage to Apache's default web root directory.

---

### Step 6: Review + Create
1. Click **Review + create** at the bottom of the screen.
2. Azure will run a validation check. Look for the green banner: `Validation passed`.
3. Click **Create**.
4. **CRITICAL POPUP:** A window titled **Generate new key pair** will appear:
   - Click **Download private key and create resource**.
   - Your browser will download `vm-web-demo_key.pem`.
   - **Do not lose this file!** It is your only cryptographic key to access your virtual machine.

---

## 5. Connecting to Your VM via SSH

Once deployment is complete (typically 1–2 minutes), click **Go to resource**. Copy your VM's **Public IP address** (e.g., `20.198.12.45`).

```
+-----------------------------------------------------------------------+
|                       HOW SSH AUTHENTICATION WORKS                    |
+-----------------------------------------------------------------------+
|  Your Laptop (Client)                     Azure VM (Cloud Server)     |
|                                                                       |
|  [ vm-web-demo_key.pem ]                 [ ~/.ssh/authorized_keys ]   |
|   (Private Key - SECRET)                  (Public Key - Stored on VM) |
|            |                                           |              |
|            +==== Encrypted Challenge Handshake =======>+              |
|                                                                       |
|  If math matches -> Access Granted as 'azureuser'!                    |
+-----------------------------------------------------------------------+
```

### Option A: Connecting from macOS / Linux / Git Bash

1. Open your terminal and move to the directory where the `.pem` file was saved:
   ```bash
   cd ~/Downloads
   ```

2. Lock down the key permissions. SSH will refuse to use a key if it is readable by other users:
   ```bash
   chmod 400 vm-web-demo_key.pem
   ```

3. Connect to the VM:
   ```bash
   ssh -i vm-web-demo_key.pem azureuser@<YOUR_VM_PUBLIC_IP>
   ```
   *(Example: `ssh -i vm-web-demo_key.pem azureuser@20.198.12.45`)*

4. When prompted: `Are you sure you want to continue connecting (yes/no/[fingerprint])?`, type `yes` and press Enter.

---

### Option B: Connecting from Windows (PowerShell)

Windows 10 and 11 have an OpenSSH client built-in:

1. Open **PowerShell** as your standard user.
2. Navigate to your Downloads folder:
   ```powershell
   cd C:\Users\<YourUsername>\Downloads
   ```
3. Run the SSH command directly:
   ```powershell
   ssh -i .\vm-web-demo_key.pem azureuser@<YOUR_VM_PUBLIC_IP>
   ```

*(If Windows warns about private key permissions, you can use the Azure Portal's built-in "Bastion" or native PowerShell `icacls` to restrict permissions).*

---

## 6. Manual Web Server Installation (Alternative Lab)

If you did not use the **Custom Data** script during deployment, you can manually install either Apache or Nginx once logged into the SSH terminal:

### Installing Apache Manually:
```bash
# Update repository lists
sudo apt update -y

# Install Apache2
sudo apt install -y apache2

# Start and enable the service
sudo systemctl start apache2
sudo systemctl enable apache2

# Check status (should show 'active (running)')
sudo systemctl status apache2

# Add custom homepage
echo "<html><body style='font-family:sans-serif;text-align:center;'><h1>Welcome to Apache on Azure!</h1><p>Configured manually via SSH</p></body></html>" | sudo tee /var/www/html/index.html
```

### Installing Nginx Manually:
```bash
# Update repository lists
sudo apt update -y

# Install Nginx
sudo apt install -y nginx

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Add custom homepage
echo "<h1>Welcome to Nginx Web Server on Azure VM!</h1>" | sudo tee /var/www/html/index.html
```

---

## 7. Testing Your Website in a Web Browser

1. Copy the **Public IP address** of your VM from the Azure Portal.
2. Open any web browser (Chrome, Firefox, Edge).
3. Type: `http://<YOUR_VM_PUBLIC_IP>` into the address bar.
   > **Note:** Do NOT type `https://` because SSL certificates are not configured yet. Use plain `http://`.
4. You will see:
   ```
   Welcome to Apache Web Server on Azure VM!
   ```

---

## 8. Troubleshooting: Network Security Groups (NSGs)

If your browser shows `Site can't be reached` or keeps loading indefinitely, the Network Security Group is blocking incoming traffic.

```
Incoming Internet Traffic: Port 80 (HTTP)
                    |
                    v
    +-------------------------------+
    | Network Security Group (NSG)  |
    |  Rule 100: Allow SSH (22)     |  ---> ALLOWED
    |  Rule 200: Allow HTTP (80)    |  ---> ALLOWED (Traffic passes!)
    |  Rule 65000: Deny All Inbound |  ---> BLOCKED (Default drop)
    +-------------------------------+
```

### How to add an NSG Inbound Rule:
1. In the Azure Portal, open your Virtual Machine page.
2. In the left navigation bar, under **Networking**, click **Network settings** (or **Networking**).
3. Check the **Inbound port rules** table. If port `80` is missing:
4. Click **Add inbound port rule** on the right side:
   - **Source:** `Any`
   - **Source port ranges:** `*`
   - **Destination:** `Any`
   - **Service:** `HTTP` (or set Destination port ranges to `80`)
   - **Protocol:** `TCP`
   - **Action:** `Allow`
   - **Priority:** `300` (Lower number = higher priority)
   - **Name:** `Allow-HTTP-80`
5. Click **Add**. The rule activates within 15–30 seconds. Refresh your browser page to confirm.

---

## 9. Virtual Machine Lifecycle & Cost Management

Understanding VM states is essential to prevent unexpected Azure billing:

| VM State | What is happening? | CPU & Memory Allocated? | Are you billed for Compute? |
| :--- | :--- | :--- | :--- |
| **Running** | The OS is booted and executing workloads. | Yes | **YES (Full Rate)** |
| **Stopped** (Shutdown inside OS) | `sudo poweroff` executed inside Linux. | Yes (Hardware remains reserved!) | **YES (Still billed!)** |
| **Stopped (deallocated)** | Stopped via Azure Portal or Azure CLI. | **No** (Hardware released back to pool). | **NO (Compute is FREE)** |
| **Deleted** | All resources permanently destroyed. | No | **NO** |

> [!WARNING]
> If you run `sudo shutdown -h now` inside your VM, Azure shows the status as **Stopped**. You are **STILL BILLED** for the CPU and RAM allocation. Always stop VMs from the **Azure Portal** (click the **Stop** button) or run `az vm deallocate` so the state reads **Stopped (deallocated)**!

*(Note: In the deallocated state, you only pay pennies per month for the persistent storage of your OS disk).*

---

## 10. Essential Azure CLI Commands for VMs

Manage your virtual machines directly from the command line:

```bash
# 1. Login to your Azure account
az login

# 2. List all virtual machines in your subscription
az vm list --output table

# 3. Create a new VM with Custom Data via CLI
az vm create \
  --resource-group rg-compute-demo \
  --name vm-cli-demo \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data @myscript.sh \
  --public-ip-sku Standard

# 4. Get the Public IP of a specific VM
az vm show -d -g rg-compute-demo -n vm-web-demo --query publicIps -o tsv

# 5. Stop and DEALLOCATE a VM (stops billing!)
az vm deallocate --resource-group rg-compute-demo --name vm-web-demo

# 6. Start a deallocated VM
az vm start --resource-group rg-compute-demo --name vm-web-demo

# 7. Restart a running VM
az vm restart --resource-group rg-compute-demo --name vm-web-demo

# 8. Delete the VM and its disks cleanly
az vm delete --resource-group rg-compute-demo --name vm-web-demo --yes
```

---

## 11. Module Summary & Key Takeaways

1. **Azure VMs are Infrastructure as a Service (IaaS):** You have root access, OS flexibility, and responsibility for patching and security.
2. **Use B-Series for Learning:** `Standard_B1s` provides sufficient power for labs at minimal cost.
3. **Use SSH Key Authentication:** Avoid passwords for public-facing cloud instances.
4. **Automate with Custom Data:** Use cloud-init scripts to configure web servers, applications, and dependencies automatically during initial VM boot.
5. **Always Deallocate:** Check that the VM status displays **Stopped (deallocated)** before closing your practice session to avoid burning subscription credits.
