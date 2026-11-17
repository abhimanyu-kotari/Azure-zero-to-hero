# 03 - High Availability: Azure VMSS & Load Balancer

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `01:44:39` to `02:29:44`

---

## 1. The Problem with Single Virtual Machines

In the previous module, we deployed a web server on a single Azure VM. While that works for small experiments, running production workloads on a single VM presents major risks:

1. **Single Point of Failure (SPOF):** If that physical host hardware fails, or during scheduled maintenance, your website goes down completely.
2. **Inability to Handle Traffic Spikes:** If 50,000 visitors arrive at once, a single VM's CPU and RAM become saturated, crashing the app.
3. **No Automatic Healing:** If the operating system crashes at 3:00 AM, nobody is awake to restart it.

To solve this, modern cloud architecture relies on two key technologies:
- **Azure Load Balancer:** Distributes incoming user traffic evenly across multiple servers.
- **Azure Virtual Machine Scale Sets (VMSS):** Automatically adds or removes VMs based on traffic demand (Elastic Scaling).

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
|            | (Routes to healthy instances)         |                  |
|            v                                       v                  |
|   +-----------------+                     +-----------------+         |
|   |   VMSS Node 1   |                     |   VMSS Node 2   |         |
|   | Apache (Port 80)|                     | Apache (Port 80)|         |
|   +-----------------+                     +-----------------+         |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 2. Scaling Concepts: Vertical vs. Horizontal

Before setting up scale sets, every cloud engineer must understand the difference between **Scaling Up** and **Scaling Out**:

```
VERTICAL SCALING (Scale Up)          HORIZONTAL SCALING (Scale Out)
     +-------------+                     +-----+  +-----+  +-----+
     |             |                     | VM1 |  | VM2 |  | VM3 |
     |  Bigger VM  |                     +-----+  +-----+  +-----+
     |  16 vCPU    |                                 +
     |  64 GB RAM  |                     Add more identical VMs
     +-------------+                     behind a Load Balancer!
```

| Metric | Vertical Scaling (Scale Up / Down) | Horizontal Scaling (Scale Out / In) |
| :--- | :--- | :--- |
| **How it works** | Resizing an existing VM to a larger size (e.g., from `B1s` to `D4s`). | Deploying *more* identical VMs side-by-side. |
| **Downtime** | **Requires a reboot** (Causes downtime while hardware resizes). | **Zero downtime** (Traffic flows to existing servers while new ones spin up). |
| **Upper Limit** | Limited by the largest physical machine Azure manufactures. | Virtually limitless (Thousands of VMs can run together). |
| **Fault Tolerance**| Low (Still only one machine; if it dies, everything dies). | **Very High** (If one VM dies, others continue serving traffic). |

---

## 3. What is an Azure Load Balancer?

An **Azure Load Balancer** operates at Layer 4 of the OSI model (Transport Layer - TCP/UDP). It distributes network traffic across healthy backend virtual machines.

### Key Components of an Azure Load Balancer:

```
+--------------------------------------------------------------------------+
|                     LOAD BALANCER COMPONENT FLOW                         |
+--------------------------------------------------------------------------+
|  1. FRONTEND IP CONFIGURATION:                                           |
|     The single public IP or private IP that clients connect to.          |
|                                                                          |
|  2. HEALTH PROBE:                                                        |
|     Constantly pings VMs (e.g., every 5s on Port 80).                    |
|     Unhealthy VMs are immediately removed from rotation.                 |
|                                                                          |
|  3. LOAD BALANCING RULES:                                                |
|     Defines how traffic is mapped (e.g., Frontend Port 80 -> Backend 80).|
|                                                                          |
|  4. BACKEND POOL:                                                        |
|     The collection of target VMs or VMSS nodes handling the requests.    |
+--------------------------------------------------------------------------+
```

---

## 4. What is a Virtual Machine Scale Set (VMSS)?

An **Azure Virtual Machine Scale Set (VMSS)** lets you deploy and manage a group of identical, load-balanced VMs. 
- The number of VM instances can automatically increase or decrease in response to demand or a defined schedule.
- VMSS provides **High Availability** across Availability Zones.
- If one instance becomes unhealthy, VMSS automatically terminates it and spins up a fresh replacement.

---

## 5. Hands-on Lab: Deploying Load-Balanced VMSS with Apache

In this lab from the video, we deploy a Load Balancer and a VMSS where every instance automatically boots with an Apache web server serving a custom webpage.

### Step 1: Create a Resource Group
1. Open the [Azure Portal](https://portal.azure.com).
2. Create a new Resource Group named `rg-vmss-demo` in your preferred region (e.g., `East US`).

---

### Step 2: Create the Virtual Machine Scale Set
1. Search for **Virtual machine scale sets** in the top search bar and click **+ Create**.
2. **Basics Tab:**
   - **Resource group:** `rg-vmss-demo`
   - **Virtual machine scale set name:** `vmss-web-cluster`
   - **Region:** `East US`
   - **Availability zone:** Select Zones `1, 2, 3` (for multi-zone redundancy)
   - **Orchestration mode:** **Uniform** (optimized for large-scale, identical stateless workloads)
   - **Security type:** `Standard`
   - **Image:** `Ubuntu Server 22.04 LTS - x64 Gen2`
   - **Size:** `Standard_B1s` (Cheapest for practice)
   - **Authentication type:** `Password` or `SSH public key`
     - Username: `azureuser`
     - Password: `AzurePassword123!` (or generate SSH key)

---

### Step 3: Configure Networking & Attach Load Balancer
1. Go to the **Networking** tab.
2. **Virtual network:** Create new or select default (`vnet-vmss-demo`).
3. Under **Load balancing**, select:
   - **Load balancing options:** `Azure load balancer`
   - **Select a load balancer:** `Create a load balancer`
   - **Load balancer name:** `lb-web-frontend`
   - **Type:** `Public`
   - **Create a backend pool:** Name it `backend-pool-web`

---

### Step 4: Configure Scaling Rules
1. Go to the **Scaling** tab.
2. **Initial instance count:** `2` (starts with 2 VMs).
3. **Scaling policy:** Select **Custom**.
   - **Minimum number of VMs:** `1`
   - **Maximum number of VMs:** `5`
   - **Scale out condition:** Metric = `CPU utilization`, Threshold = `75%`, Duration = `5 mins`, Action = `Increase count by 1`.
   - **Scale in condition:** Metric = `CPU utilization`, Threshold = `25%`, Duration = `5 mins`, Action = `Decrease count by 1`.

---

### Step 5: Advanced Tab — Inject Startup Script (Custom Data)
1. Navigate to the **Advanced** tab.
2. In the **Custom data** text box, paste the automated Apache setup script:

```bash
#!/bin/bash
sudo apt update -y
sudo apt install -y apache2
sudo systemctl start apache2
sudo systemctl enable apache2

# Display the unique hostname so we know WHICH VM instance answered our request!
INSTANCE_ID=$(hostname)
echo "<html><body style='font-family: Arial; text-align: center; margin-top: 50px;'>" | sudo tee /var/www/html/index.html
echo "<h1>Welcome to Apache on Azure VMSS!</h1>" | sudo tee -a /var/www/html/index.html
echo "<p>Served by Instance Hostname: <b>$INSTANCE_ID</b></p>" | sudo tee -a /var/www/html/index.html
echo "</body></html>" | sudo tee -a /var/www/html/index.html
```

> **Why `$INSTANCE_ID`?** When you refresh the Load Balancer IP, you will see the hostname toggle between instances, proving the Load Balancer is distributing requests!

---

### Step 6: Review + Create
1. Click **Review + create** -> **Create**.
2. Wait 3–4 minutes for the deployment to finish.

---

### Step 7: Configure Load Balancer Inbound Rule & Health Probe
By default, the Load Balancer needs an explicit rule to forward web traffic to the backend pool:

1. In the Azure Portal, go to your new Load Balancer: `lb-web-frontend`.
2. Under **Settings**, click **Health probes** -> **+ Add**:
   - **Name:** `hp-http-80`
   - **Protocol:** `TCP`
   - **Port:** `80`
   - **Interval:** `5` seconds
   - **Unhealthy threshold:** `2` consecutive failures
   - Click **Add**.
3. Under **Settings**, click **Load balancing rules** -> **+ Add**:
   - **Name:** `rule-http-80`
   - **Frontend IP address:** Select your Public IP.
   - **Backend pool:** Select `backend-pool-web`.
   - **Protocol:** `TCP`
   - **Port:** `80`
   - **Backend port:** `80`
   - **Health probe:** Select `hp-http-80`.
   - Click **Add**.

---

## 6. Testing the Load Balancer & High Availability

### Verification 1: Accessing the Web Application
1. On the Load Balancer **Overview** page, copy the **Public IP address**.
2. Paste it in your browser: `http://<LOAD_BALANCER_PUBLIC_IP>`.
3. You will see:
   ```
   Welcome to Apache on Azure VMSS!
   Served by Instance Hostname: vmss-web-cluster_0
   ```
4. Open an Incognito window or refresh using `Ctrl + F5`. You will see requests routed to `vmss-web-cluster_1`!

---

### Verification 2: Simulating Server Failure (Self-Healing Test)
1. Go to **Virtual machine scale sets** -> `vmss-web-cluster`.
2. In the left menu, click **Instances**.
3. Select `vmss-web-cluster_0` and click **Stop (Deallocate)**.
4. Refresh your browser immediately:
   - Your website **DOES NOT FAIL**.
   - The Load Balancer health probe detects node 0 is offline and immediately directs 100% of user traffic to node 1!
   - When node 0 is powered back on, the health probe passes and resumes routing traffic to it.

---

## 7. Azure Load Balancer: Basic vs. Standard SKU

| Feature | Basic SKU (Legacy) | Standard SKU (Recommended) |
| :--- | :--- | :--- |
| **Backend Pool Size** | Up to 300 instances | Up to 1,000 instances |
| **Availability Zones** | Not supported | **Zone-redundant & Zonal supported** |
| **Security by Default** | Open by default (Insecure) | **Closed by default** (Requires explicit NSG rules) |
| **Health Probes** | TCP, HTTP | TCP, HTTP, **HTTPS** |
| **SLA** | None | **99.99% uptime guarantee** |

---

## 8. Summary Checklist
- [x] Single VMs are vulnerable to outages; use VMSS and Load Balancers for production.
- [x] Horizontal scaling (adding more instances) provides zero downtime and fault tolerance.
- [x] Load Balancers distribute traffic via Frontend IPs, Backend Pools, and Health Probes.
- [x] Health Probes continuously test backend server health and pull failed nodes from rotation.
- [x] VMSS auto-scales instances up and down dynamically using metric thresholds like CPU usage.
