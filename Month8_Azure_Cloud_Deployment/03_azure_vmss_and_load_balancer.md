# 03 - Azure VMSS & Load Balancer

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `01:44:39` to `02:29:44`

---

## 1. Why a Single Virtual Machine is Not Enough

In real-world applications (like Amazon, Netflix, or food delivery apps), running your website on a single Virtual Machine is dangerous:
1. **Single Point of Failure (SPOF):** If that single physical host crashes or undergoes scheduled maintenance, your website goes down completely.
2. **Traffic Spikes:** If 50,000 customers visit your app during a festival sale, one VM will run out of CPU and RAM and crash.
3. **No Automatic Recovery:** You would have to wake up at 3:00 AM, log into the Azure Portal, and manually launch a larger server.

To solve this, modern cloud engineering relies on two services working hand-in-hand:
- **Azure Virtual Machine Scale Sets (VMSS):** Automatically adds or removes VM copies based on demand.
- **Azure Load Balancer:** Evenly distributes incoming customer traffic across all those VMs so no single server gets overwhelmed.

---

## 2. Vertical vs. Horizontal Scaling

When web traffic grows, you have two ways to add more computing power:

```
        VERTICAL SCALING (SCALE UP)              HORIZONTAL SCALING (SCALE OUT)
      Make the same server bigger                     Add more identical servers

           +---------------+                     +-------+   +-------+   +-------+
           |               |                     |  VM1  |   |  VM2  |   |  VM3  |
           |   Giant VM    |                     +-------+   +-------+   +-------+
           |  (64 Cores)   |                         ^           ^           ^
           +---------------+                         +-----------+-----------+
                   ^                                             |
           +---------------+                           [ Azure Load Balancer ]
           |  Original VM  |
           |   (2 Cores)   |
           +---------------+
```

| Dimension | Vertical Scaling (Scale Up / Down) | Horizontal Scaling (Scale Out / In) |
| :--- | :--- | :--- |
| **Concept** | Upgrading a single VM to a larger size (e.g., from 2 CPU/4 GB RAM to 16 CPU/64 GB RAM). | Adding more identical VM instances behind a load balancer. |
| **Downtime** | **Requires a reboot** (server downtime during resize). | **Zero downtime** (new VMs join the pool seamlessly). |
| **Limits** | Limited by the maximum physical server size Azure sells. | Virtually **unlimited** (can scale to 1,000+ VM instances). |
| **Fault Tolerance** | Zero (if that one big machine crashes, you are offline). | High (if 1 VM dies, the remaining 9 VMs handle traffic). |
| **Cost** | Expensive large instances running 24/7. | Cost-effective: spin up extra VMs only during rush hours. |

---

## 3. What is Azure Virtual Machine Scale Sets (VMSS)?

**Azure Virtual Machine Scale Sets (VMSS)** is an Azure compute resource that lets you deploy and manage a group of identical, load-balanced virtual machines.

### Key Features of VMSS:
1. **Identical Instances:** Every VM runs the exact same OS image, packages, code, and configuration.
2. **Autoscaling Rules:**
   - **Scale Out Rule:** *If average CPU across all VMs > 75% for 5 minutes, automatically add 2 more VMs.*
   - **Scale In Rule:** *If average CPU drops below 25%, automatically remove 2 VMs to save money.*
   - **Schedule-based:** *Scale out to 10 VMs every Monday morning at 9:00 AM, and scale down to 2 VMs on Friday evening.*
3. **High Availability Built-in:**
   - **Fault Domains (FD):** Physical server racks with separate power and network switches. VMSS spreads your VMs across different racks so a blown fuse doesn't kill all your servers.
   - **Update Domains (UD):** Logical groups of VMs. During routine Azure platform patching, Azure reboots only one UD at a time, keeping your application online.

---

## 4. What is Azure Load Balancer?

An **Azure Load Balancer** operates at **Layer 4 of the OSI model (Transport Layer - TCP/UDP)**. It acts as the "traffic police officer" standing in front of your servers:

```
                          [ Internet Users ]
                                  |
                                  v
                    [ Azure Public Load Balancer ]
                      (Single Public IP: Port 80)
                                  |
         +------------------------+------------------------+
         |                        |                        |
         v                        v                        v
    [ VMSS - VM 1 ]          [ VMSS - VM 2 ]          [ VMSS - VM 3 ]
    (Private IP: .4)         (Private IP: .5)         (Private IP: .6)
```

### The 4 Core Components of a Load Balancer:
1. **Frontend IP Configuration:** The single public IP address (or internal private IP) that users connect to.
2. **Backend Pool:** The list of virtual machines (or VMSS instances) that receive the traffic and run your application code.
3. **Health Probe:** Continuously tests each VM (e.g., sends an HTTP request to port 80 every 5 seconds):
   - If VM 2 crashes or stops responding, the health probe marks it "Unhealthy".
   - The Load Balancer **immediately stops sending traffic to VM 2**, routing users only to healthy VMs (VM 1 and VM 3).
4. **Load Balancing Rules:** Defines how incoming traffic is routed. For example: *Traffic arriving on Frontend Port 80 (TCP) should be forwarded to Backend Pool instances on Port 80.*

---

## 5. Public Load Balancer vs. Internal Load Balancer

- **Public Load Balancer:**
  - Has a public internet-facing IP address.
  - Used for external web traffic (routing internet users to frontend web servers).
- **Internal (Private) Load Balancer:**
  - Has a private IP address within your Azure Virtual Network (VNet).
  - Used for internal tiers (e.g., routing web frontend servers to private backend API servers or databases).

---

## 6. Hands-on Walkthrough: Creating a Load-Balanced Web App

In the video, M Prashant demonstrates the complete setup:

### Step 1: Create the Azure Load Balancer
1. Search for **Load balancers** in the Azure Portal -> Click **Create**.
2. Resource Group: `rg-vmss-demo`.
3. Name: `my-web-loadbalancer`.
4. Region: e.g., *East US*.
5. Type: **Public**.
6. SKU: **Standard** (recommended for production and security).
7. Frontend IP Configuration: Click *Add frontend IP configuration*, select *Public IP*, create a new one named `lb-public-ip`.
8. Click **Review + Create**.

### Step 2: Configure Health Probe & Load Balancing Rule
1. Go to your new Load Balancer resource.
2. Under **Settings**, click **Health probes** -> **+ Add**:
   - Name: `http-health-probe`
   - Protocol: `TCP`
   - Port: `80`
   - Interval: `5` seconds.
3. Under **Settings**, click **Load balancing rules** -> **+ Add**:
   - Name: `http-lb-rule`
   - Frontend IP: select `lb-public-ip`.
   - Protocol: `TCP`, Port: `80`, Backend Port: `80`.
   - Health Probe: select `http-health-probe`.
   - Click **Add**.

### Step 3: Deploy VMSS Connected to the Load Balancer
1. Search for **Virtual machine scale sets** -> Click **Create**.
2. Name: `my-frontend-vmss`.
3. Image: `Ubuntu Server 22.04 LTS`.
4. Size: `Standard_B1s`.
5. Under **Networking**, choose **Use an Azure load balancer**:
   - Select `my-web-loadbalancer`.
   - Select the backend pool.
6. Under **Scaling**:
   - Initial instance count: `2`.
   - Scaling policy: *Autoscale* (Min: 2, Max: 5, Scale out when CPU > 75%).
7. Under **Advanced**, paste a cloud-init script in **Custom data** to install Nginx automatically on boot.
8. Click **Review + create**.

### Step 4: Testing Failover
1. Once deployed, copy the **Frontend Public IP** of the Load Balancer.
2. Paste it in your web browser. You will see the Nginx welcome page.
3. If you manually stop VM 1 in the Azure Portal, the Load Balancer detects the failure within 10 seconds and routes 100% of your requests to VM 2 without any downtime or error page!

---

## 7. Summary Checklist
- [x] Vertical scaling resizes one machine (causes downtime); Horizontal scaling adds identical machines (zero downtime).
- [x] VMSS automates deploying and autoscaling identical VMs based on CPU or schedules.
- [x] Azure Load Balancer operates at Layer 4 (TCP/UDP) distributing traffic across a backend pool.
- [x] Health Probes continuously monitor instances and automatically bypass dead or crashed servers.
