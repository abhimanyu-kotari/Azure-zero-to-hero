# 02 - Compute: Azure Virtual Machines (VMs) Masterclass

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `00:51:43` to `01:44:39` (Full 53-Minute Deep Dive)

---

## 1. What is Azure Compute?

In cloud computing, **Compute** refers to the raw physical processing power—CPU cores, Random Access Memory (RAM), storage disks, and networking controllers—required to run your software, execute algorithms, serve APIs, and host websites.

Microsoft Azure categorizes compute services along a spectrum from maximum administrative control to maximum automation:

```
+---------------------------------------------------------------------------------------------------+
|                                     AZURE COMPUTE SPECTRUM                                        |
+---------------------------------------------------------------------------------------------------+
|  IaaS (Infrastructure as a Service)  <------------------------->  PaaS / Serverless (Fully Managed) |
|                                                                                                   |
|  [ Azure VM ]      --->  [ VMSS ]        --->  [ AKS ]        --->  [ ACA ]       ---> [ Functions ]|
|  Full OS Control         Identical Cluster     Kubernetes           Microservices      Event-Driven |
|  You patch OS            Auto-scaling          Orchestration        Scale-to-Zero      Pure Code    |
+---------------------------------------------------------------------------------------------------+
```

The fundamental compute building block of Microsoft Azure is the **Azure Virtual Machine (VM)**.

---

## 2. Anatomy & Architecture of an Azure Virtual Machine

An **Azure Virtual Machine** is an on-demand, scalable computing instance running inside Microsoft's enterprise datacenters. When you create a VM, Azure does not simply create an isolated software box; it orchestrates and links several independent Azure cloud resources together:

```
+---------------------------------------------------------------------------------------+
|                         AZURE VIRTUAL MACHINE ARCHITECTURE                            |
+---------------------------------------------------------------------------------------+
|                                                                                       |
|   +-------------------------------------------------------------------------------+   |
|   |                         VIRTUAL MACHINE INSTANCE                              |   |
|   |  - Host Hardware: Compute Processor (vCPU) + Volatile Memory (RAM)            |   |
|   |  - OS: Ubuntu Server 22.04 LTS / Windows Server 2022                          |   |
|   |  - Identity: System-assigned or User-assigned Managed Identity                |   |
|   +---------------------------------------+---------------------------------------+   |
|                                           |                                           |
|                                           v                                           |
|   +-------------------------------------------------------------------------------+   |
|   |                              STORAGE LAYER                                    |   |
|   |  1. OS Disk: Stores operating system files, kernel, and software (Standard/   |   |
|   |     Premium SSD, e.g., 30 GB to 128 GB).                                      |   |
|   |  2. Temporary / Ephemeral Disk: Local scratch space (lost on deallocation).   |   |
|   |  3. Data Disks (Optional): Additional VHDs attached via LUNs for databases.   |   |
|   +---------------------------------------+---------------------------------------+   |
|                                           |                                           |
|                                           v                                           |
|   +-------------------------------------------------------------------------------+   |
|   |                          NETWORKING LAYER (NIC)                               |   |
|   |  - Network Interface Card (NIC): Attaches VM to the Virtual Network           |   |
|   |  - Private IP: Internal address (e.g., 10.0.0.4) for subnet communication     |   |
|   |  - Accelerated Networking (SR-IOV): Up to 30 Gbps ultra-low latency           |   |
|   +-------------------+-----------------------------------+-----------------------+   |
|                       |                                   |                           |
|                       v                                   v                           |
|   +---------------------------------------+   +-----------------------------------+   |
|   |            PUBLIC IP ADDRESS          |   |   NETWORK SECURITY GROUP (NSG)    |   |
|   |  - Routable over the global internet  |   |  - Stateful perimeter firewall    |   |
|   |  - Dynamic vs. Static IP assignment   |   |  - Filters ports 22, 80, 443, 3389|   |
|   |  - Optional DNS Name Label            |   |  - Priority evaluated 100 to 4096 |   |
|   +---------------------------------------+   +-----------------------------------+   |
|                                                                                       |
+---------------------------------------------------------------------------------------+
```

---

## 3. Azure VM Sizing Families & Hardware Series

Azure names its virtual machine sizes using lettered families. Choosing the wrong family will either starve your application of resources or waste thousands of dollars.

```
       B-Series             D-Series              E-Series              F-Series
    [Burstable]         [General Purpose]    [Memory Optimized]   [Compute Optimized]
     1 vCPU / 1GB         2 vCPU / 8GB          2 vCPU / 16GB         8 vCPU / 16GB
      Dev / Test          Web & APIs             DBs & Caches           Rendering
```

| Series | Category | vCPU : RAM Ratio | Best Real-World Use Cases |
| :--- | :--- | :--- | :--- |
| **B-Series** | **Burstable (Economical)** | 1:1 to 1:4 | **Development, testing, student learning, personal websites, build agents**. Uses CPU credits. |
| **D-Series** | **General Purpose** | 1:4 (Balanced) | Production web servers, e-commerce APIs, small-to-medium databases, enterprise apps. |
| **E-Series** | **Memory Optimized** | 1:8 (High RAM) | Relational database engines (SQL Server, PostgreSQL), in-memory caches (Redis), SAP HANA, heavy analytics. |
| **F-Series** | **Compute Optimized** | 1:2 (High CPU) | High-volume batch processing, data analytics algorithms, video encoding, gaming server hosting. |
| **L-Series** | **Storage Optimized** | High NVMe I/O | NoSQL databases (Cassandra, MongoDB), data warehousing requiring extreme local read/write IOPS. |
| **N-Series** | **GPU Accelerated** | NVIDIA GPUs | Machine learning training, deep neural networks, AI inference, 3D CAD rendering, visual effects. |
| **M-Series** | **Extreme Memory** | Up to 12 TB RAM | Massive enterprise in-memory databases and mission-critical ERPs. |

### Deep Dive: How the B-Series (Burstable) Works
The **B-Series** (e.g., `Standard_B1s`, `Standard_B2s`) is what M Prashant uses throughout the video:
- **Baseline Performance:** A `B1s` instance has a baseline CPU performance of 10% of a vCPU.
- **Earning Credits:** While your VM is idle (under 10% CPU), it banks **CPU credits**.
- **Bursting:** When users hit your website or you run `apt update`, the VM consumes its banked credits to burst up to 100% of the vCPU at full speed!
- **Ideal For:** Student practice and low-traffic workloads where the server is idle 90% of the day.

---

## 4. Availability Options: SLA, Zones, and Sets

Before clicking create, Azure asks how your VM should be protected against datacenter failures:

```
+-----------------------------------------------------------------------------------+
|                           AZURE HIGH AVAILABILITY SPECTRUM                        |
+-----------------------------------------------------------------------------------+
|  Single VM with Premium SSD           ---> 99.9% SLA   (Basic hardware guarantee) |
|  Availability Set (Fault/Update Doms) ---> 99.95% SLA  (Datacenter rack safety)   |
|  Availability Zones (Zone 1, 2, 3)    ---> 99.99% SLA  (Multi-datacenter safety)  |
|  Multi-Region Disaster Recovery       ---> 99.999% SLA (Survives regional disaster)|
+-----------------------------------------------------------------------------------+
```

### 1. Availability Zones (AZs)
- Physically separated datacenter buildings within the same Azure region.
- Each zone possesses **independent power, cooling, and fiber networking**.
- Choosing **Availability Zone 1, 2, or 3** guarantees that if one datacenter suffers a catastrophic flood or power failure, your VMs in other zones remain operational.
- Delivers an industry-leading **99.99% uptime SLA**.

### 2. Availability Sets
Used when an Azure region does not support Availability Zones:
- **Fault Domains (FD):** Represents physical server racks sharing a single power supply unit and network switch. Azure distributes your VMs across 2 or 3 Fault Domains so a tripped circuit breaker only affects one rack.
- **Update Domains (UD):** Logical groups of underlying hardware that can be rebooted concurrently during platform OS maintenance. Azure splits VMs across up to 20 Update Domains to guarantee zero downtime during scheduled Microsoft patching.
- Delivers a **99.95% uptime SLA**.

---

## 5. Security Types: Standard vs. Trusted Launch vs. Confidential

| Security Type | Architecture | Protection Provided |
| :--- | :--- | :--- |
| **Standard** | Standard virtualization. | Baseline hypervisor isolation. |
| **Trusted Launch** | **Secure Boot + Virtual TPM (vTPM)**. | Prevents kernel-level rootkits, bootkits, and unauthorized firmware tampering. (Recommended default in 2026). |
| **Confidential VM** | **Hardware memory encryption (AMD SEV-SNP / Intel TDX)**. | Encrypts VM RAM in hardware. Even Microsoft datacenter engineers and hypervisor admins cannot read memory contents! |

---

## 6. OS Images & Generation 1 vs. Generation 2

When selecting an operating system image (Ubuntu, Red Hat, Windows Server):
- **Generation 1 (Legacy):** Uses legacy BIOS booting, MBR partitions, and emulated IDE controllers. Limited to 2 TB OS disks.
- **Generation 2 (Modern Standard):** Uses **UEFI firmware**, GPT partition tables, PCIe-based virtual hardware, and supports OS disks greater than 2 TB, faster boot times, and Trusted Launch.

---

## 7. Step-by-Step Portal Walkthrough: Creating a Linux VM

Follow this exact field-by-field tutorial to deploy your first cloud server:

### Step 1: Open Virtual Machine Creation
1. Sign in to the [Azure Portal](https://portal.azure.com).
2. In the top search bar, type `Virtual machines` and click **Virtual machines**.
3. Click **+ Create** -> **Azure virtual machine**.

---

### Step 2: Tab 1 — "Basics" Configuration

```
+-----------------------------------------------------------------------------------+
| BASICS TAB CONFIGURATION                                                          |
+-----------------------------------------------------------------------------------+
| Project Details:                                                                  |
|   Subscription:          [ Your Active Subscription ]                             |
|   Resource Group:        [ Create new: rg-compute-demo ]                          |
|                                                                                   |
| Instance Details:                                                                 |
|   Virtual Machine Name:  [ vm-web-demo ]                                          |
|   Region:                [ (US) East US / Central India ]                         |
|   Availability Options:  [ No infrastructure redundancy required ] (For learning) |
|   Security Type:         [ Standard ] or [ Trusted launch ]                       |
|   Image:                 [ Ubuntu Server 22.04 LTS - x64 Gen2 ]                   |
|   VM Architecture:       [ x64 ]                                                  |
|   Run with Azure Spot:   [ Leave unchecked ]                                      |
|   Size:                  [ Standard_B1s (1 vCPU, 1 GiB memory) ]                  |
|                                                                                   |
| Administrator Account:                                                            |
|   Authentication Type:   [ SSH public key ]  <-- Always use for Linux!            |
|   Username:              [ azureuser ]                                            |
|   SSH Public Key Source: [ Generate new key pair ]                                |
|   Key Pair Name:         [ vm-web-demo_key ]                                      |
|                                                                                   |
| Inbound Port Rules:                                                               |
|   Public Inbound Ports:  [ Allow selected ports ]                                 |
|   Select Inbound Ports:  [x] SSH (22)   [x] HTTP (80)   [ ] HTTPS (443)           |
+-----------------------------------------------------------------------------------+
```

> [!IMPORTANT]
> **Why SSH Keys over Passwords?**  
> Public-facing cloud servers that accept passwords are bombarded with automated dictionary attacks. An RSA 2048/4096-bit cryptographic key pair is virtually impossible to brute-force.

---

### Step 3: Tab 2 — "Disks" Configuration

```
+-----------------------------------------------------------------------------------+
| DISKS TAB CONFIGURATION                                                           |
+-----------------------------------------------------------------------------------+
| OS Disk:                                                                          |
|   OS Disk Type:          [ Standard SSD LRS ] (Cost-effective for learning)       |
|   Delete with VM:        [x] (Ensures the disk is deleted when the VM is deleted) |
|   Key Management:        [ Platform-managed key (Default) ]                       |
|   Enable Ultra Disk:     [ Unchecked ]                                            |
|                                                                                   |
| Data Disks:                                                                       |
|   Attach a new disk:     (Optional - used when extra storage beyond OS is needed) |
+-----------------------------------------------------------------------------------+
```

#### Comparison of Azure Disk Types:
- **Ultra Disk:** Sub-millisecond latency, up to 160,000 IOPS. For heavy Oracle/SAP workloads.
- **Premium SSD v2:** Configurable IOPS and throughput independently at low cost.
- **Premium SSD:** Baseline enterprise production disk for high IOPS.
- **Standard SSD (Our Choice):** Ideal for dev/test, learning, and web servers with low traffic.
- **Standard HDD:** Magnetic spinning drives; slowest performance, cheapest backup storage.

---

### Step 4: Tab 3 — "Networking" Configuration

```
+-----------------------------------------------------------------------------------+
| NETWORKING TAB CONFIGURATION                                                      |
+-----------------------------------------------------------------------------------+
| Network Interface:                                                                |
|   Virtual Network:       [ Create new or select: vnet-compute-demo ]              |
|   Subnet:                [ default (10.0.0.0/24) ]                                |
|   Public IP:             [ Create new: vm-web-demo-ip ]                           |
|   NIC Network Security:  [ Basic ]                                                |
|   Public Inbound Ports:  [ Allow selected ports: SSH (22), HTTP (80) ]            |
|   Delete Public IP & NIC:[x] Check both! (Prevents orphaned billing!)             |
|   Accelerated Networking:[ Enabled if supported by VM size ]                      |
|   Load Balancing:        [ None ]                                                 |
+-----------------------------------------------------------------------------------+
```

---

### Step 5: Tab 4 — "Management" Configuration
- **Microsoft Defender for Cloud:** Free tier enabled.
- **Identity:** `System-assigned managed identity` (Optional: enable if your VM needs to access Azure Storage or Key Vault without hardcoded credentials).
- **Auto-shutdown (Crucial for Students!):**
  - Check `[x] Enable auto-shutdown`.
  - Shutdown time: `07:00 PM` (19:00).
  - Time zone: Select your local time zone.
  - Email notification: Enter your email address to receive an alert before shutdown.
  - *Why?* This prevents leaving a VM running over the weekend and burning through your free subscription credits!

---

### Step 6: Tab 5 — "Monitoring" Configuration
- **Boot diagnostics:** Select **Enable with managed storage account (recommended)**.
  - *Why?* If the Linux kernel crashes or GRUB fails to boot, Boot Diagnostics captures console serial logs and screenshots so you can troubleshoot!

---

### Step 7: Tab 6 — "Advanced" Configuration (Automated Custom Data Script)

> [!IMPORTANT]
> In the video, M Prashant explains that rather than manually logging into the VM to install packages, we can inject an automated bootstrap script into the **Custom data** field. Azure passes this script to `cloud-init`, which executes with root privileges the very first time the VM boots!

Scroll down to **Custom data** and paste the exact script shown in the video:

```bash
#!/bin/bash
sudo apt update -y
sudo apt install -y apache2
sudo systemctl start apache2
sudo systemctl enable apache2

echo "<html><h1>Welcome to Apache Web Server on Azure VM!</h1></html>" | sudo tee /var/www/html/index.html
```

#### Line-by-Line Breakdown:
1. `#!/bin/bash`: Shebang line telling the Linux kernel to execute this file using the Bash interpreter.
2. `sudo apt update -y`: Refreshes the local package index against Ubuntu software mirrors.
3. `sudo apt install -y apache2`: Installs the Apache HTTP daemon; `-y` automatically answers "yes" to all confirmation prompts.
4. `sudo systemctl start apache2`: Boots the Apache web service into memory.
5. `sudo systemctl enable apache2`: Configures systemd to automatically restart Apache if the VM is ever rebooted.
6. `echo "..." | sudo tee /var/www/html/index.html`: Overwrites the default Apache splash page with our custom welcome banner.

---

### Step 8: Review + Create & Download Private Key
1. Click **Review + create**.
2. Wait for the green banner: `Validation passed`.
3. Check the pricing estimate at the bottom (e.g., ~$0.0104/hour).
4. Click **Create**.
5. **CRITICAL STEP:** A popup window titled **Generate new key pair** will appear:
   - Click **Download private key and create resource**.
   - Your browser will save `vm-web-demo_key.pem`.
   - Store this file safely in your `Downloads` or `~/.ssh/` directory! If you lose this file, you cannot log in via SSH!
6. Azure provisions the VM in 1 to 2 minutes.

---

## 8. Connecting to Your VM: The 6 Methods (Portal Connect & Cloud Shell)

Once the VM deployment is complete, click **Go to resource**. Notice the top action bar on the VM Overview page featuring the **Connect** button:

```
+-----------------------------------------------------------------------------------+
| AZURE VM OVERVIEW TOP BAR                                                         |
| [ Connect v ]  [ Restart ]  [ Stop ]  [ Capture ]  [ Delete ]  [ Refresh ]        |
|      |                                                                            |
|      +---> [ Native SSH ]         ---> Standard terminal connection               |
|      +---> [ Azure Cloud Shell ]  ---> In-browser terminal with uploaded key     |
|      +---> [ Azure Bastion ]      ---> Browser TLS proxy without public IP        |
|      +---> [ Serial Console ]     ---> Out-of-band emergency hardware terminal    |
|      +---> [ Run Command ]        ---> Execute scripts without logging in         |
+-----------------------------------------------------------------------------------+
```

---

### Method 1: Local Terminal / Git Bash / macOS (Native SSH)

1. Open your terminal and navigate to your key's location:
   ```bash
   cd ~/Downloads
   ```

2. **Fix File Permissions (Mandatory!):**
   SSH clients reject private keys that have broad read/write permissions:
   ```bash
   chmod 400 vm-web-demo_key.pem
   ```
   *(If you skip this step, SSH throws the error: `Permissions 0644 for 'key.pem' are too open. It is required that your private key files are NOT accessible by others!`)*

3. **Initiate the SSH Session:**
   ```bash
   ssh -i vm-web-demo_key.pem azureuser@<YOUR_VM_PUBLIC_IP>
   ```
   *(Example: `ssh -i vm-web-demo_key.pem azureuser@20.120.45.67`)*

4. When prompted: `Are you sure you want to continue connecting (yes/no/[fingerprint])?`, type `yes` and hit Enter.
5. You are now logged into the Ubuntu command line:
   ```
   azureuser@vm-web-demo:~$
   ```

---

### Method 2: Azure Cloud Shell (Browser-Based Terminal)

> [!TIP]
> In the video, M Prashant highlights **Azure Cloud Shell** as an incredible feature for users who cannot or do not want to install SSH clients or terminal emulators on their local machines!

Azure Cloud Shell is an interactive, authenticated, browser-accessible terminal for managing Azure resources.

```
+-----------------------------------------------------------------------------------+
|                           AZURE CLOUD SHELL WORKFLOW                              |
+-----------------------------------------------------------------------------------+
|  1. Click [ >_ ] Cloud Shell icon in the Azure Portal top navigation bar.         |
|  2. Select [ Bash ] environment.                                                  |
|  3. Click [ Upload/Download ] icon in Cloud Shell top bar -> [ Upload ].          |
|  4. Select 'vm-web-demo_key.pem' from your local computer.                       |
|  5. Run: chmod 400 vm-web-demo_key.pem                                            |
|  6. Run: ssh -i vm-web-demo_key.pem azureuser@<YOUR_VM_PUBLIC_IP>                  |
|  ===> You are instantly connected to your VM inside your browser!                 |
+-----------------------------------------------------------------------------------+
```

#### Step-by-Step Walkthrough in Portal:
1. In the top navigation bar of the Azure Portal, click the **Cloud Shell icon (`>_`)** (next to the search bar and notifications bell).
2. **First-time setup:** If prompted, select **Bash** and click **Create storage** (Azure attaches a small Azure Files share to persist your `$HOME` directory files).
3. At the top of the Cloud Shell pane, click the **Upload/Download files icon** (two vertical arrows) -> click **Upload**.
4. Browse your local files and select `vm-web-demo_key.pem`.
5. Verify the key was uploaded:
   ```bash
   ls -la vm-web-demo_key.pem
   ```
6. Secure the key permissions inside Cloud Shell:
   ```bash
   chmod 400 vm-web-demo_key.pem
   ```
7. Connect to your VM directly from Cloud Shell:
   ```bash
   ssh -i vm-web-demo_key.pem azureuser@<YOUR_VM_PUBLIC_IP>
   ```
8. Type `yes` when prompted. You now have full terminal access inside the Azure Portal browser window!

---

### Method 3: Windows PowerShell & Command Prompt

Windows 10 and 11 feature an OpenSSH client pre-installed:

1. Open **PowerShell** as your standard user.
2. Navigate to your Downloads folder:
   ```powershell
   cd C:\Users\<YourUsername>\Downloads
   ```
3. Connect using the SSH command:
   ```powershell
   ssh -i .\vm-web-demo_key.pem azureuser@<YOUR_VM_PUBLIC_IP>
   ```

#### Fixing Windows Key Permission Errors (ACLs):
If Windows complains about key security permissions, run these PowerShell commands to reset file inheritance:
```powershell
icacls.exe .\vm-web-demo_key.pem /reset
icacls.exe .\vm-web-demo_key.pem /grant:r "$($env:USERNAME):(R)"
icacls.exe .\vm-web-demo_key.pem /inheritance:r
```

---

### Method 4: PuTTY for Windows (Legacy Connection)
1. Download and open **PuTTYgen**.
2. Click **Load** -> select `All Files (*.*)` -> choose `vm-web-demo_key.pem`.
3. Click **Save private key** -> save as `vm-web-demo_key.ppk`.
4. Open **PuTTY**:
   - Host Name: `azureuser@<YOUR_VM_PUBLIC_IP>`
   - In left menu: **Connection** -> **SSH** -> **Auth** -> **Credentials**.
   - Browse and select your `.ppk` file.
   - Click **Open**.

---

### Method 5: Azure Serial Console (Emergency Out-of-Band Access)

What if you accidentally broke the SSH daemon configuration, or the VM's Network Security Group completely blocks port 22, or the network interface card fails?

Azure provides the **Serial Console**:
- Connects directly to the virtual machine's **COM1 serial port** via hypervisor hardware emulation!
- **Requires zero network connectivity**—no public IP, no private IP, and no NSG rules are needed.
- Shows raw Linux kernel boot messages, systemd startup logs, and kernel panic stack traces.

#### How to use:
1. In the Azure Portal, open your Virtual Machine page.
2. In the left navigation menu, scroll down to **Help** -> click **Serial console**.
3. Press **Enter** in the console window.
4. Enter your Linux username (`azureuser`) and password to access the emergency shell!

---

### Method 6: Azure Run Command (Execute Scripts without SSH)

Azure allows administrators to execute bash or PowerShell scripts on a running VM directly from the portal or Azure CLI via the background **Azure VM Guest Agent**:

1. In the left navigation menu of your VM, scroll to **Operations** -> click **Run command**.
2. Select **RunShellScript**.
3. Type any bash script (e.g., `sudo systemctl status apache2 && df -h`).
4. Click **Run**. Azure executes the script with root privileges and outputs the result in the portal window!

---

## 9. Comprehensive Port & Connectivity Testing Lab (Commands Used After Connecting)

> [!IMPORTANT]
> In the masterclass video, right after connecting to the VM via Azure Cloud Shell, M Prashant demonstrates a complete sequence of diagnostic commands to verify:
> 1. Is the web server process running?
> 2. Is port 80 actually listening on all network interfaces?
> 3. Does the server respond to internal HTTP loopback requests?
> 4. Can an external client (like Azure Cloud Shell or a browser) reach port 80 across the Azure Network Security Group?
> 5. How can you observe live incoming visitor traffic in the access logs?

Follow this exact step-by-step verification lab:

```
+-----------------------------------------------------------------------------------+
|                        END-TO-END CONNECTIVITY VERIFICATION FLOW                  |
+-----------------------------------------------------------------------------------+
|                                                                                   |
|  STEP 1: Check Internal Process        --->  sudo systemctl status apache2        |
|  STEP 2: Check Listening Ports (80)    --->  sudo netstat -tulnp | grep :80       |
|  STEP 3: Test Internal Loopback        --->  curl -I http://localhost             |
|  STEP 4: Test External Reachability    --->  nc -zv <PUBLIC_IP> 80 (from outside) |
|  STEP 5: Test External HTTP Response   --->  curl -I http://<PUBLIC_IP>           |
|  STEP 6: Observe Real-Time Access Logs --->  sudo tail -f /var/log/apache2/access.log
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

---

### Step 1: Verify the Web Server Process Status
The first step after logging in via SSH is checking if systemd successfully started the web server daemon:

```bash
sudo systemctl status apache2
```

**Expected Healthy Terminal Output:**
```text
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2026-11-17 10:15:22 UTC; 2min 14s ago
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 1842 (apache2)
      Tasks: 55 (limit: 1114)
     Memory: 5.1M
        CPU: 48ms
     CGroup: /system.slice/apache2.service
             ├─1842 /usr/sbin/apache2 -k start
             ├─1844 /usr/sbin/apache2 -k start
             └─1845 /usr/sbin/apache2 -k start
```
- Look for **`Active: active (running)`** highlighted in green.
- If it shows `inactive (dead)` or `failed`, start it using:
  ```bash
  sudo systemctl start apache2
  ```

---

### Step 2: Check if Port 80 is Actively Listening (`netstat` & `ss`)
A service can be running, but is it actually bound to network port `80`?

#### Option A: Using `netstat`
```bash
sudo netstat -tulnp | grep :80
```
**Expected Output:**
```text
tcp   0   0 0.0.0.0:80   0.0.0.0:*   LISTEN   1842/apache2
```

#### Understanding the `netstat -tulnp` Flags:
| Flag | Meaning | What it tells you |
| :---: | :--- | :--- |
| `-t` | **TCP** | Displays TCP socket connections. |
| `-u` | **UDP** | Displays UDP socket connections. |
| `-l` | **Listening** | Filters exclusively for ports actively waiting for incoming connections. |
| `-n` | **Numeric** | Shows numerical port `80` instead of resolving the service name `http`. |
| `-p` | **Program/PID** | Reveals the exact Process ID (e.g., `1842`) and executable name (`apache2`). |
| `0.0.0.0:80` | **All IPv4 Interfaces** | The web server is listening on every NIC and IP address attached to the VM! |

#### Option B: Using Modern `ss` (Socket Statistics)
On modern Ubuntu systems where `net-tools` is not installed by default, use `ss`:
```bash
sudo ss -tulpn | grep :80
```
**Expected Output:**
```text
tcp   LISTEN   0   511   0.0.0.0:80   0.0.0.0:*   users:(("apache2",pid=1845,fd=4),("apache2",pid=1844,fd=4),("apache2",pid=1842,fd=4))
```

#### Option C: Using `lsof` (List Open Files)
```bash
sudo lsof -i :80
```
**Expected Output:**
```text
COMMAND   PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
apache2  1842     root    4u  IPv4  24891      0t0  TCP *:http (LISTEN)
apache2  1844 www-data    4u  IPv4  24891      0t0  TCP *:http (LISTEN)
apache2  1845 www-data    4u  IPv4  24891      0t0  TCP *:http (LISTEN)
```

---

### Step 3: Test Local HTTP Response from INSIDE the VM (`curl` Loopback)
Before testing from the outside internet, confirm that the local operating system can talk to Apache over the loopback interface (`127.0.0.1` / `localhost`):

```bash
# 1. Fetch the raw HTML content
curl http://localhost
```
**Expected Output:**
```html
<html><h1>Welcome to Apache Web Server on Azure VM!</h1></html>
```

```bash
# 2. Inspect the HTTP Response Headers (The -I flag sends a HEAD request)
curl -I http://localhost
```
**Expected Output:**
```text
HTTP/1.1 200 OK
Date: Sun, 17 Nov 2026 10:18:04 GMT
Server: Apache/2.4.52 (Ubuntu)
Last-Modified: Sun, 17 Nov 2026 10:14:50 GMT
ETag: "40-62720d43a6ec0"
Accept-Ranges: bytes
Content-Length: 64
Content-Type: text/html
```
- **`HTTP/1.1 200 OK`**: Proves the web server is operating flawlessly!

---

### Step 4: Test Port 80 Reachability from OUTSIDE the VM (`nc` Netcat)

Now exit your SSH session (or open a **second tab in Azure Cloud Shell** or your local computer terminal):

```bash
# Testing from your local computer terminal or Azure Cloud Shell (OUTSIDE the VM):
nc -zv <YOUR_VM_PUBLIC_IP> 80
```
*(Example: `nc -zv 20.120.45.67 80`)*

#### Understanding the Result:
- **If Port 80 IS OPEN in Azure NSG:**
  ```text
  Connection to 20.120.45.67 80 port [tcp/http] succeeded!
  ```
  *(Traffic successfully traversed the Azure virtual datacenter firewall!)*

- **If Port 80 IS BLOCKED in Azure NSG:**
  ```text
  nc: connect to 20.120.45.67 port 80 (tcp) failed: Connection timed out
  ```
  *(The Azure Network Security Group dropped the packets before they reached your VM!)*

---

### Step 5: Test External HTTP Request (`curl` from Outside)

From your local machine or Azure Cloud Shell:
```bash
curl -I http://<YOUR_VM_PUBLIC_IP>
```
**Expected Output:**
```text
HTTP/1.1 200 OK
Date: Sun, 17 Nov 2026 10:20:15 GMT
Server: Apache/2.4.52 (Ubuntu)
Content-Type: text/html
```

---

### Step 6: Watch Live Incoming Visitor Traffic (The Access Log Stream)

In the video, M Prashant demonstrates how to visually prove that incoming web browser traffic is reaching the server:

1. In your VM SSH terminal, run `tail -f` to lock onto the Apache access log file:
   ```bash
   sudo tail -f /var/log/apache2/access.log
   ```

2. Now open your web browser or run `curl http://<YOUR_VM_PUBLIC_IP>` from your laptop.
3. Look at your VM terminal in real time:
   ```text
   49.37.152.88 - - [17/Nov/2026:10:21:42 +0000] "GET / HTTP/1.1" 200 482 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
   ```
   - **`49.37.152.88`**: Your laptop's real public IP address.
   - **`"GET / HTTP/1.1"`**: The browser requesting the root index page.
   - **`200`**: HTTP success status code.
   - Press `Ctrl + C` to stop watching the log.

---

## 10. Essential Linux Commands Used in the VM Shell

Once connected to your virtual machine shell, here is the complete, categorized reference of the essential Linux commands demonstrated in the video:

### 1. Identity & System Information Commands
```bash
# Check who you are logged in as (prints 'azureuser')
whoami

# Print current working directory (prints '/home/azureuser')
pwd

# Check system hostname and private IP address
hostname
hostname -I

# View complete Linux kernel and OS distribution details
uname -a
cat /etc/os-release

# Switch to the root superuser account
sudo -i
# (or return to azureuser with: exit)
```

---

### 2. Package Management Commands
```bash
# Refresh package repository lists against mirrors
sudo apt update -y

# Upgrade all installed packages to their latest security patches
sudo apt upgrade -y

# Install essential networking and troubleshooting utilities
sudo apt install -y curl wget git net-tools htop
```

---

### 3. Service & Process Management (systemd)
```bash
# Check if the Apache web server service is running
sudo systemctl status apache2

# Start the web server service
sudo systemctl start apache2

# Stop the web server service
sudo systemctl stop apache2

# Restart the service (useful after editing configuration files)
sudo systemctl restart apache2

# Enable the service to automatically boot upon VM reboot
sudo systemctl enable apache2

# Inspect the active running processes in the system
ps aux | grep apache2

# Monitor real-time CPU, RAM, and process activity
top
# (Press 'q' to exit top)
```

---

### 4. Networking & Port Verification Commands
```bash
# Check listening TCP/UDP ports (Verify if port 80 is listening!)
sudo netstat -tuln
# Expected output: tcp 0 0 0.0.0.0:80 0.0.0.0:* LISTEN

# Modern alternative to netstat (Socket Statistics)
sudo ss -tulpn

# Test the web server locally from INSIDE the VM terminal (Sub-millisecond loopback)
curl http://localhost
curl -I http://127.0.0.1
# Expected output: HTTP/1.1 200 OK

# Inspect private IP configuration and MAC addresses on network interfaces
ip addr show
# (Look for interface 'eth0' and inet '10.0.0.4')
```

---

### 5. Disk, Filesystem & Memory Monitoring Commands
```bash
# Check available and used disk space on all mounted filesystems (Human-readable)
df -h
# Expected output: /dev/root mounted on / (shows size, used, and available space)

# Check system RAM and swap memory usage in Megabytes or Gigabytes
free -m
free -h

# List all attached block storage devices and partitions
lsblk
```

---

### 6. Real-Time Log Inspection & Troubleshooting Commands
```bash
# View the execution output of the Custom Data (cloud-init) startup script
cat /var/log/cloud-init-output.log

# View the detailed cloud-init system log
sudo cat /var/log/cloud-init.log

# Stream live incoming website visitor traffic in real time!
sudo tail -f /var/log/apache2/access.log
# (As users visit your Public IP in their browser, requests will log here live!)

# View web server error logs
sudo tail -f /var/log/apache2/error.log

# View systemd journal logs specifically for the Apache daemon
sudo journalctl -u apache2 --no-pager -n 50
```

---

### 7. Web Content & File Management Commands
```bash
# Check the contents of the Apache web root directory
ls -la /var/www/html/

# Inspect the current index.html file
cat /var/www/html/index.html

# Overwrite index.html with a custom welcome page
echo "<html><h1>Welcome to Apache Web Server on Azure VM!</h1></html>" | sudo tee /var/www/html/index.html
```

---

## 11. Manual Web Server Installation (Alternative Lab)

If you deployed the VM without the **Custom Data** script, you can install and configure web servers manually after logging into the shell:

### Installing Apache Manually:
```bash
# 1. Update package lists
sudo apt update -y

# 2. Install Apache2
sudo apt install -y apache2

# 3. Check if service is active and running
sudo systemctl status apache2

# 4. Create custom webpage
echo "<html><body style='font-family: Arial; text-align: center; margin-top: 50px;'><h1>Welcome to Apache on Azure!</h1><p>Manually installed via SSH</p></body></html>" | sudo tee /var/www/html/index.html
```

### Installing Nginx Manually (Alternative Web Server):
```bash
# 1. Update package lists
sudo apt update -y

# 2. Install Nginx
sudo apt install -y nginx

# 3. Start and enable service
sudo systemctl start nginx
sudo systemctl enable nginx

# 4. Verify status
sudo systemctl status nginx

# 5. Create custom homepage
echo "<h1>Welcome to Nginx on Azure VM!</h1>" | sudo tee /var/www/html/index.html
```

---

## 12. Testing Your Website in a Web Browser

1. Go to the Azure Portal -> your VM overview page -> copy the **Public IP address**.
2. Open any web browser (Chrome, Edge, Firefox).
3. Type: `http://<YOUR_VM_PUBLIC_IP>` in the address bar.
   > **CRITICAL:** Do NOT type `https://`! Because we have not yet installed an SSL certificate, typing `https://` will fail. You must use plain `http://`.
4. You will immediately see the custom page:
   ```
   Welcome to Apache Web Server on Azure VM!
   ```

---

## 13. Troubleshooting: Network Security Groups (NSGs)

What if you paste the IP address into your browser and the page spins endlessly with `Site can't be reached`?

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

### Diagnostic Checklist:
1. **Is Port 80 open on Azure NSG?**
   - Go to Azure Portal -> VM -> **Networking** (or **Network settings**).
   - Look at the **Inbound port rules** table.
   - If port `80` is not listed, click **+ Add inbound port rule**:
     - Source: `Any`
     - Source port ranges: `*`
     - Destination: `Any`
     - Service: `HTTP` (Port `80`)
     - Protocol: `TCP`
     - Action: `Allow`
     - Priority: `200` (Lower numbers execute first)
     - Name: `Allow-HTTP-80`
   - Click **Add**. Wait 20 seconds and refresh your browser.

2. **Is Apache running inside the VM?**
   ```bash
   sudo systemctl status apache2
   ```
   If it shows `inactive` or `dead`, start it:
   ```bash
   sudo systemctl restart apache2
   ```

3. **Did the Custom Data script fail?**
   Inspect the cloud-init execution logs to see if a command failed during boot:
   ```bash
   cat /var/log/cloud-init-output.log
   ```

4. **Is Ubuntu OS Firewall (UFW) blocking port 80?**
   ```bash
   sudo ufw status
   # If active, allow HTTP:
   sudo ufw allow 80/tcp
   ```

---

## 14. Making Public IPs Static & Creating a DNS Name

By default, Azure Public IPs are **Dynamic**: if you stop and deallocate your VM, the Public IP is released back to Azure's global pool. When you start it back up, it receives a *different* IP address!

### How to Assign a Static Public IP:
1. In the Azure Portal, open your VM's Overview page.
2. Click on the **Public IP address** hyperlink.
3. In the left menu under **Settings**, click **Configuration**.
4. Change assignment from **Dynamic** to **Static**.
5. **DNS name label (Optional but awesome!):**
   - Enter a unique name, e.g., `myazurewebdemo-prashant`.
   - Click **Save**.
   - You now have a permanent free domain URL:  
     `http://myazurewebdemo-prashant.eastus.cloudapp.azure.com`!

---

## 15. Attaching, Partitioning & Mounting Data Disks in Linux

In production, you never store application databases on the OS disk. You attach a dedicated **Data Disk**.

### Step 1: Attach Disk in Azure Portal
1. Open your VM -> under **Settings**, click **Disks**.
2. Under **Data disks**, click **+ Create and attach a new disk**:
   - Disk name: `datadisk-01`
   - Size: `32 GiB` (Standard SSD)
   - Host caching: `Read/write`
3. Click **Apply**.

---

### Step 2: Initialize & Mount the Disk inside Linux
1. SSH into your VM and list all block devices:
   ```bash
   lsblk
   ```
   *(You will see your OS disk `sda`, temporary disk `sdb`, and the new raw disk `sdc`).*

2. Partition the disk using `fdisk`:
   ```bash
   sudo fdisk /dev/sdc
   # Type 'n' for new partition, press Enter 3 times for defaults, then 'w' to write changes!
   ```

3. Format the partition with the `ext4` filesystem:
   ```bash
   sudo mkfs.ext4 /dev/sdc1
   ```

4. Create a mount point and mount it:
   ```bash
   sudo mkdir -p /mnt/datadrive
   sudo mount /dev/sdc1 /mnt/datadrive
   ```

5. **Configure Persistent Mount on Boot (`/etc/fstab`):**
   If you do not configure `/etc/fstab`, the disk unmounts upon reboot!
   ```bash
   # Find the UUID of the disk
   sudo blkid /dev/sdc1
   # Output: /dev/sdc1: UUID="1234-abcd-5678-ef01" TYPE="ext4"

   # Add to /etc/fstab:
   echo "UUID=1234-abcd-5678-ef01 /mnt/datadrive ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab
   ```

---

## 16. Virtual Machine Lifecycle & Cost Management

Understanding VM states is essential to prevent surprise cloud bills:

| VM State | What is happening? | CPU & Memory Allocated? | Billed for Compute? |
| :--- | :--- | :--- | :--- |
| **Running** | Operating system is active and processing workloads. | Yes | **YES (Full Rate)** |
| **Stopped** (Inside OS) | `sudo poweroff` executed inside Linux terminal. | **Yes** (Host hardware remains locked!) | **YES (Still billed!)** |
| **Stopped (deallocated)** | Stopped via Azure Portal or Azure CLI. | **No** (Hardware released back to pool). | **NO (Compute is $0)** |
| **Deleted** | All resources permanently destroyed. | No | **NO** |

> [!CAUTION]
> If you shut down your VM by running `sudo shutdown -h now` inside the terminal, the Azure Portal status reads **Stopped**. Azure **STILL CHARGES YOU** for the reserved CPU and memory! You must click **Stop** in the Azure Portal or run `az vm deallocate` until the status explicitly reads **Stopped (deallocated)**!

*(Note: In the deallocated state, you only pay a nominal fee of ~$1.50/month for persistent storage of the OS disk).*

---

## 17. Essential Azure CLI Commands for VMs

Manage your virtual machines directly from the command line or terminal:

```bash
# 1. Login to Azure
az login

# 2. List all VMs in your subscription
az vm list --output table

# 3. Create a new VM with Custom Data via CLI
az vm create \
  --resource-group rg-compute-demo \
  --name vm-cli-demo \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data @myscript.sh \
  --public-ip-sku Standard

# 4. Open Port 80 on VM firewall via CLI
az vm open-port --resource-group rg-compute-demo --name vm-cli-demo --port 80 --priority 200

# 5. Query the public IP of a VM
az vm show -d -g rg-compute-demo -n vm-web-demo --query publicIps -o tsv

# 6. Run a shell script directly from Azure Portal/CLI without SSH!
az vm run-command invoke \
  --resource-group rg-compute-demo \
  --name vm-web-demo \
  --command-id RunShellScript \
  --scripts "sudo systemctl status apache2"

# 7. Resize a VM to a larger size
az vm resize --resource-group rg-compute-demo --name vm-web-demo --size Standard_B2s

# 8. Stop and DEALLOCATE a VM (Halts compute billing!)
az vm deallocate --resource-group rg-compute-demo --name vm-web-demo

# 9. Start a deallocated VM
az vm start --resource-group rg-compute-demo --name vm-web-demo

# 10. Permanently delete VM and attached disks
az vm delete --resource-group rg-compute-demo --name vm-web-demo --yes
```

---

## 18. Module Summary & Student Checklist
- [x] Azure VMs provide Infrastructure as a Service (IaaS) with full root OS control.
- [x] B-Series (Burstable) uses CPU credits and is ideal for budget-friendly learning.
- [x] Availability Zones deliver 99.99% SLA across separate physical datacenters.
- [x] Always authenticate with SSH public keys rather than passwords for Linux.
- [x] Use **Custom Data** (cloud-init) scripts in the Advanced tab to automate server setup at boot.
- [x] Network Security Groups (NSGs) act as firewalls; open port 80 for web traffic.
- [x] Make Public IPs static to prevent address changes upon VM reboot.
- [x] Attach separate Data Disks for databases and configure `/etc/fstab` using UUIDs.
- [x] **Always Deallocate:** Ensure VM status reads **Stopped (deallocated)** when not practicing to eliminate compute charges.
