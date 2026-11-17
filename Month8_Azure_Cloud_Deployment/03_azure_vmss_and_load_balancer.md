# 03 - High Availability: Azure VMSS & Load Balancer

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `01:44:39` to `02:29:44`

---

## 1. The Problem with Single Virtual Machines

In Module 02, we deployed an Apache web server on a single standalone virtual machine. While great for learning, running production workloads on a single VM introduces critical vulnerabilities:

1. **Single Point of Failure (SPOF):** If the physical host hardware fails, or if Microsoft performs host hypervisor patching, your entire website goes offline.
2. **Inability to Handle Traffic Surges:** If 100,000 users suddenly arrive during a flash sale, a single VM's CPU and RAM become completely saturated, causing the server to freeze and drop user connections.
3. **No Automatic Recovery:** If the web service crashes at 3:00 AM, there is no automated mechanism to restart or replace the server.

To achieve enterprise **High Availability (HA)** and **Scalability**, modern cloud architectures combine two services:
- **Azure Load Balancer:** Sits in front of your servers to evenly distribute incoming traffic and detect failed instances.
- **Azure Virtual Machine Scale Sets (VMSS):** Automatically deploys, configures, and dynamically scales a cluster of identical VMs based on real-time traffic demand.

```
+-----------------------------------------------------------------------+
|                    HIGH AVAILABILITY ARCHITECTURE                     |
+-----------------------------------------------------------------------+
|                                                                       |
|                          Internet Users                               |
|                                |                                      |
|                                v                                      |
|                   +-------------------------+                         |
|                   |   AZURE LOAD BALANCER   |                         |
|                   |  Frontend Public IP     |                         |
|                   +------------+------------+                         |
|                                |                                      |
|            +-------------------+-------------------+                  |
|            | (Routes only to healthy instances)    |                  |
|            v                                       v                  |
|   +-----------------+                     +-----------------+         |
|   |   VMSS Node 0   |                     |   VMSS Node 1   |         |
|   | Apache (Port 80)|                     | Apache (Port 80)|         |
|   +-----------------+                     +-----------------+         |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 2. Scaling Architectures: Vertical vs. Horizontal

Before configuring scale sets, every cloud engineer must know when to scale **Up** versus when to scale **Out**:

```
VERTICAL SCALING (Scale Up)          HORIZONTAL SCALING (Scale Out)
     +-------------+                     +-----+  +-----+  +-----+
     |             |                     | VM0 |  | VM1 |  | VM2 |
     |  Bigger VM  |                     +-----+  +-----+  +-----+
     |  16 vCPU    |                                 +
     |  64 GB RAM  |                     Add more identical VMs
     +-------------+                     behind a Load Balancer!
```

| Metric | Vertical Scaling (Scale Up / Down) | Horizontal Scaling (Scale Out / In) |
| :--- | :--- | :--- |
| **How it works** | Resizing an existing VM to a larger hardware size (e.g., from `B1s` to `D4s`). | Adding *more* identical VMs side-by-side to share the workload. |
| **Downtime Impact** | **Requires a reboot** (Causes application downtime while resizing). | **Zero downtime** (New instances join the pool seamlessly). |
| **Upper Limit** | Hard ceiling: limited by the largest physical server Azure manufactures. | Virtually unlimited: thousands of VMs can run concurrently. |
| **Fault Tolerance**| Poor (Still only one server; if it fails, the application is down). | **Excellent** (If one node fails, healthy nodes continue serving users). |
| **Cost Efficiency** | Expensive (Big VM sizes carry exponential hourly rates). | Highly cost-effective (Only pay for extra VMs during peak hours). |

---

## 3. How Azure Load Balancer Works (Layer 4)

An **Azure Load Balancer** operates at **Layer 4 (Transport Layer - TCP/UDP)** of the OSI model. It does not inspect HTTP URLs or cookies; it strictly forwards raw network packets based on IP address and port numbers.

### Core Components of a Load Balancer:

```
+--------------------------------------------------------------------------+
|                     LOAD BALANCER COMPONENT FLOW                         |
+--------------------------------------------------------------------------+
|  1. FRONTEND IP CONFIGURATION:                                           |
|     The public IP address (or internal private IP) that clients connect  |
|     to when visiting your website.                                       |
|                                                                          |
|  2. HEALTH PROBE:                                                        |
|     Regularly tests each backend VM (e.g., every 5 seconds on Port 80).  |
|     If a VM stops answering, the probe flags it unhealthy and removes it |
|     from routing immediately.                                            |
|                                                                          |
|  3. LOAD BALANCING RULES:                                                |
|     Defines how incoming traffic is forwarded.                           |
|     Example: "Forward traffic from Frontend Port 80 to Backend Port 80". |
|                                                                          |
|  4. BACKEND POOL:                                                        |
|     The collection of target VMs or VMSS instances ready to receive      |
|     incoming user traffic.                                               |
+--------------------------------------------------------------------------+
```

---

## 4. What is a Virtual Machine Scale Set (VMSS)?

An **Azure Virtual Machine Scale Set (VMSS)** allows you to create and manage a group of load-balanced, identical VMs:
- **Centralized Management:** You define a single VM model (OS image, hardware size, startup script), and Azure deploys identical copies.
- **Dynamic Autoscaling:** Azure automatically provisions new VMs when CPU utilization exceeds a target (e.g., > 75%) and deletes excess VMs when traffic subsides.
- **Multi-Zone Redundancy:** Distributes VM instances evenly across Azure Availability Zones (Zone 1, Zone 2, Zone 3) to survive entire datacenter failures.
- **Self-Healing:** If an instance crashes or fails health probe checks, VMSS automatically terminates it and deploys a clean replacement.

---

## 5. Hands-on Lab: Deploying Load-Balanced VMSS with Apache

In this comprehensive hands-on lab from the video, we configure a public Load Balancer and a VMSS where every instance automatically installs Apache and outputs its unique server hostname.

### Step 1: Create a Resource Group
1. Sign in to the [Azure Portal](https://portal.azure.com).
2. Search for **Resource groups** and click **+ Create**.
3. Name: `rg-vmss-demo`.
4. Region: `East US` (or your nearest region).
5. Click **Review + create** -> **Create**.

---

### Step 2: Create the Virtual Machine Scale Set

1. Search for **Virtual machine scale sets** in the top search bar and click **+ Create**.

#### Tab 1: Basics
- **Subscription:** Select your subscription.
- **Resource group:** `rg-vmss-demo`.
- **Virtual machine scale set name:** `vmss-web-cluster`.
- **Region:** `East US`.
- **Availability zone:** Check **Zone 1, 2, 3** (enables multi-zone high availability).
- **Orchestration mode:** **Uniform** (optimized for large-scale, identical stateless web servers).
- **Security type:** `Standard`.
- **Scaling mode:** `Autoscale` (or configure in the Scaling tab).
- **Image:** `Ubuntu Server 22.04 LTS - x64 Gen2`.
- **VM architecture:** `x64`.
- **Size:** Click *See all sizes* and choose `Standard_B1s` (1 vCPU, 1 GiB RAM) to minimize learning costs.
- **Administrator account:**
  - Authentication type: **Password** (or SSH key).
  - Username: `azureuser`
  - Password: `AzurePassword123!`

---

#### Tab 2: Disks
- **OS disk type:** **Standard SSD LRS** (economical, reliable).
- Leave encryption and data disks as default.

---

#### Tab 3: Networking & Attaching the Load Balancer
- **Virtual network:** Click *Create new* -> Name: `vnet-vmss-demo` (Subnet: `10.0.0.0/24`).
- **Network Interface (NIC):** Default.
- **Load balancing:**
  - Select: **Azure load balancer**.
  - Choose: **Create a load balancer**.
  - Load balancer name: `lb-web-frontend`.
  - Type: **Public**.
  - Create a backend pool: Name it `backend-pool-web`.

---

#### Tab 4: Scaling Configuration
- **Initial instance count:** `2` (starts with two active VMs).
- **Scaling policy:** Select **Custom**.
  - **Minimum number of VMs:** `1`
  - **Maximum number of VMs:** `5`
  - **Scale out condition (Add a VM):**
    - Metric: `Percentage CPU`
    - Threshold: `75%`
    - Duration: `5 minutes`
    - Action: `Increase count by 1`
    - Cooldown: `5 minutes`
  - **Scale in condition (Remove a VM):**
    - Metric: `Percentage CPU`
    - Threshold: `25%`
    - Duration: `5 minutes`
    - Action: `Decrease count by 1`
    - Cooldown: `5 minutes`

---

#### Tab 5: Advanced Tab — Startup Script (Custom Data)
Scroll down to the **Custom data** input field and paste the startup script:

```bash
#!/bin/bash
# 1. Update package lists
sudo apt update -y

# 2. Install Apache web server
sudo apt install -y apache2

# 3. Start and enable Apache service
sudo systemctl start apache2
sudo systemctl enable apache2

# 4. Extract the unique instance hostname
INSTANCE_ID=$(hostname)

# 5. Build a dynamic web page that reveals which server answered!
echo "<html><body style='font-family: Arial, sans-serif; text-align: center; margin-top: 50px; background-color: #f4f7f6;'>" | sudo tee /var/www/html/index.html
echo "<h1 style='color: #0078d4;'>🚀 Azure Virtual Machine Scale Set (VMSS)</h1>" | sudo tee -a /var/www/html/index.html
echo "<h2>Welcome to High Availability on Microsoft Azure!</h2>" | sudo tee -a /var/www/html/index.html
echo "<div style='display: inline-block; padding: 20px 40px; background: white; border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); margin-top: 20px;'>" | sudo tee -a /var/www/html/index.html
echo "<p style='font-size: 18px;'>Handled by VMSS Instance Hostname:</p>" | sudo tee -a /var/www/html/index.html
echo "<h3 style='color: #d83b01; font-size: 28px; margin: 10px 0;'>$INSTANCE_ID</h3>" | sudo tee -a /var/www/html/index.html
echo "</div>" | sudo tee -a /var/www/html/index.html
echo "<p style='color: #666; margin-top: 30px;'>Refresh this page to see requests distributed across healthy instances!</p>" | sudo tee -a /var/www/html/index.html
echo "</body></html>" | sudo tee -a /var/www/html/index.html
```

#### Why `$INSTANCE_ID` is essential:
Every VM instance in a scale set receives a unique internal hostname (e.g., `vmss-web-cluster_0`, `vmss-web-cluster_1`). By injecting `$INSTANCE_ID` into the HTML, we can visually verify that the Load Balancer is distributing incoming requests across distinct servers!

---

#### Tab 6: Review + Create
1. Click **Review + create**.
2. Confirm validation shows `Validation passed`.
3. Click **Create**.
4. Deployment completes in 3–5 minutes.

---

### Step 3: Configure Load Balancer Health Probe & Inbound Rule

Once deployed, the Load Balancer needs to know:
1. How to verify that instances are alive (Health Probe).
2. What port to expose to the world (Load Balancing Rule).

#### Adding the Health Probe:
1. In the Azure Portal, open the Load Balancer: `lb-web-frontend`.
2. Under **Settings** in the left menu, click **Health probes**.
3. Click **+ Add**:
   - **Name:** `hp-http-80`
   - **Protocol:** `TCP` (or `HTTP`)
   - **Port:** `80`
   - **Path:** `/` (if HTTP selected)
   - **Interval:** `5` seconds (Pings each server every 5 seconds)
   - **Unhealthy threshold:** `2` consecutive failures (If 2 pings fail, remove the VM immediately!)
4. Click **Add**.

```
+-----------------------------------------------------------------------+
|                    HEALTH PROBE EVALUATION LOOP                       |
+-----------------------------------------------------------------------+
|  Every 5 seconds:                                                     |
|  [ Load Balancer Probe ] ===== TCP SYN (Port 80) =====> [ VM Node 0 ] |
|  [ Load Balancer Probe ] <==== TCP SYN-ACK =========== [ VM Node 0 ]  |
|  ===> Status: HEALTHY (Keep routing traffic!)                         |
|                                                                       |
|  If no answer after 2 consecutive probes:                             |
|  ===> Status: UNHEALTHY (Instantly drop from rotation!)               |
+-----------------------------------------------------------------------+
```

---

#### Adding the Load Balancing Rule:
1. In `lb-web-frontend`, click **Load balancing rules** under **Settings**.
2. Click **+ Add**:
   - **Name:** `rule-http-80`
   - **IP Version:** `IPv4`
   - **Frontend IP address:** Select your Load Balancer's public IP.
   - **Backend pool:** Select `backend-pool-web`.
   - **Protocol:** `TCP`
   - **Port:** `80` (The public port users connect to).
   - **Backend port:** `80` (The internal port Apache listens on).
   - **Health probe:** Select `hp-http-80`.
   - **Session persistence:** `None` (Requests from the same user can go to any healthy VM).
   - **Idle timeout:** `4` minutes.
3. Click **Add**.

---

## 6. Testing High Availability & Self-Healing

### Verification 1: Accessing the Application
1. In `lb-web-frontend`, open the **Overview** page.
2. Copy the **Public IP address** (e.g., `20.120.45.67`).
3. Open a browser window and navigate to:
   `http://20.120.45.67`
4. The page renders:
   ```
   🚀 Azure Virtual Machine Scale Set (VMSS)
   Handled by VMSS Instance Hostname: vmss-web-cluster_0
   ```
5. Open an Incognito window or press `Ctrl + F5` to force refresh.
6. The hostname switches to `vmss-web-cluster_1`! The Load Balancer is successfully sharing the workload between both virtual machines.

---

### Verification 2: Simulating Server Outage (Zero-Downtime Test)
What happens if one of the physical servers crashes?

1. Go to **Virtual machine scale sets** -> `vmss-web-cluster`.
2. In the left navigation menu, click **Instances**.
3. Select `vmss-web-cluster_0` and click the **Stop (Deallocate)** button at the top.
4. Return to your web browser and continuously refresh the Load Balancer IP:
   - **Result:** You experience **ZERO downtime**!
   - Within 10 seconds (2 probe intervals), the Health Probe flags node 0 as offline.
   - The Load Balancer automatically routes 100% of incoming web requests to node 1!
5. Start `vmss-web-cluster_0` back up:
   - As soon as Apache boots, the health probe passes, and node 0 automatically rejoins the active rotation!

---

### Verification 3: Stress-Testing CPU to Trigger Auto-Scaling
To observe VMSS automatically provisioning new virtual machines under load:

1. SSH into one of the VMSS instances (or connect via Azure Bastion).
2. Install the Linux `stress` utility:
   ```bash
   sudo apt update && sudo apt install -y stress
   ```
3. Run a CPU saturation test for 10 minutes:
   ```bash
   stress --cpu 2 --timeout 600s
   ```
4. Within 5 minutes, Azure Monitor detects CPU utilization exceeding the 75% threshold.
5. Check **Instances** in the Azure Portal: VMSS automatically launches `vmss-web-cluster_2` to absorb the incoming surge!

---

## 7. Troubleshooting Common VMSS & Load Balancer Issues

| Symptom | Probable Cause | Exact Fix |
| :--- | :--- | :--- |
| Browser shows `Connection timed out` on Load Balancer IP. | Missing Load Balancing rule or NSG blocking port 80. | Verify that a Load Balancing rule exists mapping port 80 to backend port 80. Ensure the NSG allows inbound TCP port 80 from `Any`. |
| All instances show `Unhealthy` in Health Probe status. | Apache failed to install or is not running on port 80. | SSH into an instance and run `sudo systemctl status apache2`. Check the custom data script logs at `/var/log/cloud-init-output.log`. |
| Requests only hit one VM and never distribute. | Browser HTTP session caching or TCP keep-alive. | Test using `curl -I http://<LOAD_BALANCER_IP>` from terminal or open multiple private/incognito windows. Set Session Persistence to `None`. |
| VMSS does not scale out during high load. | Scale-out metric duration is too long or threshold is too high. | Review the autoscale condition. Azure requires average CPU to stay above the threshold for the full duration (e.g., 5 mins) before triggering scale-out. |

---

## 8. Azure CLI Commands for VMSS & Load Balancer

```bash
# 1. List all instances in a VMSS
az vmss list-instances --resource-group rg-vmss-demo --name vmss-web-cluster -o table

# 2. Manually scale instance count to 3
az vmss scale --resource-group rg-vmss-demo --name vmss-web-cluster --new-capacity 3

# 3. Restart all instances in the scale set
az vmss restart --resource-group rg-vmss-demo --name vmss-web-cluster

# 4. Get Load Balancer public IP address
az network public-ip show --resource-group rg-vmss-demo --name lb-web-frontend-pip --query ipAddress -o tsv

# 5. Delete resource group when finished (saves budget!)
az group delete --name rg-vmss-demo --yes --no-wait
```

---

## 9. Summary Checklist
- [x] VMSS eliminates single points of failure by maintaining clusters of identical VMs.
- [x] Azure Load Balancer (Layer 4) distributes traffic across backend pools using health probes.
- [x] Health Probes continuously monitor servers and remove failed instances within seconds.
- [x] Custom Data scripts ensure every newly provisioned VMSS instance automatically boots configured with applications.
- [x] Autoscale rules dynamically scale out during high traffic and scale in during idle periods to minimize cloud spend.
