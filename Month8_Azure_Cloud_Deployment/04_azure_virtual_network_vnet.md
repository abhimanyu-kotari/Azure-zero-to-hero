# 04 - Networking: Azure Virtual Network (VNet) & Peering

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `02:29:44` to `03:10:24`

---

## 1. What is an Azure Virtual Network (VNet)?

An **Azure Virtual Network (VNet)** is your own isolated, private network in the Microsoft Azure cloud. It is the fundamental building block for your private network in Azure.

Just like a physical office network connects desktops, printers, and database servers via Ethernet cables and switches, a VNet enables Azure resources (such as Virtual Machines, AKS clusters, and Database instances) to securely communicate with:
1. Each other (internally)
2. The internet (outbound or inbound)
3. On-premises datacenters (via VPN Gateway or ExpressRoute)

```
+-------------------------------------------------------------------------+
|                       AZURE VIRTUAL NETWORK (VNet)                      |
|                           Address Space: 10.0.0.0/16                    |
+-------------------------------------------------------------------------+
|                                                                         |
|   +----------------------------+     +----------------------------+     |
|   |   Frontend Web Subnet      |     |   Backend Database Subnet  |     |
|   |        10.0.1.0/24         |     |        10.0.2.0/24         |     |
|   |                            |     |                            |     |
|   |   [ VM-Web-01 ]            |     |   [ VM-DB-01 ]             |     |
|   |   Private IP: 10.0.1.4     |     |   Private IP: 10.0.2.4     |     |
|   |   Public IP: 20.x.x.x      |     |   (No Public IP - Private!)|     |
|   +--------------+-------------+     +--------------^-------------+     |
|                  |                                  |                   |
|                  +======== Private VNet Traffic ====+                   |
|                            (Secure & Encrypted)                         |
+-------------------------------------------------------------------------+
```

---

## 2. Understanding IP Addressing & CIDR Notation

### Public vs. Private IP Addresses
- **Private IP Address:** Used exclusively *inside* your VNet. Devices on the public internet cannot see or ping this IP (e.g., `10.0.1.4`).
- **Public IP Address:** Routable across the global internet. Used when external users need to reach your web applications (e.g., `52.183.45.12`).

### RFC 1918 Private IP Ranges
Azure VNets generally use the standard private IP address ranges:
- `10.0.0.0` to `10.255.255.255` (Class A - Most common for enterprise clouds)
- `172.16.0.0` to `172.31.255.255` (Class B)
- `192.168.0.0` to `192.168.255.255` (Class C)

### Demystifying CIDR Notation (Classless Inter-Domain Routing)
CIDR is represented as `/X` appended to an IP address, representing the number of fixed network bits:

| CIDR Prefix | Total IP Addresses | Azure Usable IPs | Common Use Case |
| :--- | :--- | :--- | :--- |
| **/16** (e.g., `10.0.0.0/16`) | 65,536 | 65,531 | Typical entire Virtual Network address space |
| **/24** (e.g., `10.0.1.0/24`) | 256 | 251 | Typical Subnet size for VMs, containers, apps |
| **/26** (e.g., `10.0.3.0/26`) | 64 | 59 | Azure Bastion or Application Gateway Subnet |
| **/28** (e.g., `10.0.4.0/28`) | 16 | 11 | Small specialized service subnet |

> [!IMPORTANT]
> **The 5 Azure Reserved IPs:**  
> In every subnet you create in Azure, Microsoft automatically reserves **5 IP addresses**. If your subnet is `10.0.1.0/24` (256 total IPs):
> 1. `10.0.1.0`: Network address (Protocol requirement)
> 2. `10.0.1.1`: Default gateway (Azure uses this to route traffic out of the subnet)
> 3. `10.0.1.2`: Azure DNS mapping
> 4. `10.0.1.3`: Azure DNS secondary / internal coordination
> 5. `10.0.1.255`: Network broadcast address (Azure does not use broadcast, but reserves it)
> 
> Therefore, the first virtual machine deployed into a `10.0.1.0/24` subnet will **always** receive `10.0.1.4`!

---

## 3. Subnets & Network Security Groups (NSGs)

### What is a Subnet?
A subnet is a range of IP addresses within a VNet. Subnets let you organize and isolate different tiers of your application (e.g., Public Web Subnet vs. Private DB Subnet).

### Network Security Groups (NSGs)
An **NSG** acts as a stateful packet-filtering firewall at the subnet or network interface (NIC) level.

```
Traffic Packet Arrives: Source IP: Any, Port 80
                    |
                    v
+-------------------------------------------------------------+
|               NETWORK SECURITY GROUP EVALUATION             |
|                                                             |
| Priority 100: Allow SSH (Port 22)       -> No match, next   |
| Priority 200: Allow HTTP (Port 80)      -> MATCH! ALLOWED   |
| Priority 300: Deny All Web              -> Skipped          |
| Priority 65000: Default AllowVNet       -> Skipped          |
| Priority 65500: Default DenyAllInBound  -> Skipped          |
+-------------------------------------------------------------+
```

### Key Rules of NSGs:
1. **Priority Numbers:** Range from **100 to 4096**.
2. **Lowest Number Wins:** Rules are evaluated in strict priority order. The moment a rule matches the packet, evaluation **stops**.
3. **Default Rules:** Azure provides built-in fallback rules at priorities 65000+ that cannot be deleted:
   - `AllowVNetInBound`: Any VM in the VNet can talk to any other VM in the VNet.
   - `AllowAzureLoadBalancerInBound`: Allows Azure health probes to monitor VMs.
   - `DenyAllInBound`: Drops all other incoming traffic from the internet.

---

## 4. Connecting Networks: Azure VNet Peering

By default, Virtual Networks are completely isolated from one another. A VM in `VNet-A` cannot talk to a VM in `VNet-B`, even if they are in the same subscription!

**VNet Peering** connects two Virtual Networks seamlessly:
- Traffic flows over **Microsoft's private high-speed fiber backbone**, never traversing the public internet.
- Latency is identical to communication within a single network.
- **Two types:**
  1. **Regional VNet Peering:** Both VNets are in the same Azure region (e.g., both in `East US`).
  2. **Global VNet Peering:** VNets reside in different regions worldwide (e.g., `East US` and `West Europe`).

```
+--------------------------+                      +--------------------------+
|      VNet-1 (East US)    |                      |    VNet-2 (West Europe)  |
|   Address: 10.1.0.0/16   |                      |   Address: 10.2.0.0/16   |
|                          |                      |                          |
|    [ VM-App-1 ]          |                      |    [ VM-App-2 ]          |
|    IP: 10.1.1.4          |                      |    IP: 10.2.1.4          |
+------------+-------------+                      +-------------^------------+
             |                                                  |
             +======== Azure Global VNet Peering ===============+
                       - No Internet traversal
                       - End-to-end encrypted backbone
```

> [!WARNING]
> **No Overlapping CIDR Blocks:** VNet Peering **requires** that the two VNets have non-overlapping IP address spaces. For example, you cannot peer `10.0.0.0/16` with another `10.0.0.0/16`.

---

## 5. Hands-on Lab: Configuring VNet Peering Step-by-Step

In this lab, we prove that two isolated VNets cannot talk until Peering is enabled:

### Step 1: Create VNet-1 and VM-1
1. In Azure Portal, create a Resource Group: `rg-networking-demo`.
2. Create **Virtual Network 1**:
   - Name: `vnet-production-east`
   - Region: `East US`
   - IPv4 Address Space: `10.1.0.0/16`
   - Subnet Name: `subnet-app`, Subnet range: `10.1.1.0/24`
3. Launch **VM-1** inside `vnet-production-east`:
   - Name: `vm-east-01`
   - Private IP will be: `10.1.1.4`

---

### Step 2: Create VNet-2 and VM-2
1. Create **Virtual Network 2**:
   - Name: `vnet-analytics-east`
   - Region: `East US`
   - IPv4 Address Space: `10.2.0.0/16` *(Notice non-overlapping IP!)*
   - Subnet Name: `subnet-data`, Subnet range: `10.2.1.0/24`
2. Launch **VM-2** inside `vnet-analytics-east`:
   - Name: `vm-analytics-01`
   - Private IP will be: `10.2.1.4`

---

### Step 3: Test Initial Connectivity (Failure Expected)
1. SSH into `vm-east-01`.
2. Attempt to ping the private IP of `vm-analytics-01`:
   ```bash
   ping 10.2.1.4
   ```
3. **Result:** 100% packet loss! The networks are completely isolated.

---

### Step 4: Configure VNet Peering
1. In the Azure Portal, go to `vnet-production-east`.
2. In the left navigation menu under **Settings**, click **Peerings**.
3. Click **+ Add**:
   - **This virtual network (vnet-production-east):**
     - Peering link name: `peering-east-to-analytics`
     - Allow traffic to remote virtual network: `Allow (default)`
   - **Remote virtual network (vnet-analytics-east):**
     - Peering link name: `peering-analytics-to-east`
     - Virtual network deployment model: `Resource manager`
     - Select `vnet-analytics-east`
     - Allow traffic to remote virtual network: `Allow (default)`
4. Click **Add**. Azure will configure the bidirectional peering link in seconds.
5. The peering status will display: **Connected**.

---

### Step 5: Verify Connectivity (Success!)
1. Return to the SSH session on `vm-east-01`.
2. Run the ping command again:
   ```bash
   ping -c 4 10.2.1.4
   ```
3. **Result:** Success! Packets are answered with sub-millisecond latency:
   ```
   64 bytes from 10.2.1.4: icmp_seq=1 ttl=64 time=0.812 ms
   64 bytes from 10.2.1.4: icmp_seq=2 ttl=64 time=0.745 ms
   ```

---

## 6. Azure CLI Commands for Networking

```bash
# Create a Resource Group
az group create --name rg-networking-demo --location eastus

# Create Virtual Network 1
az network vnet create \
  --resource-group rg-networking-demo \
  --name vnet-production-east \
  --address-prefixes 10.1.0.0/16 \
  --subnet-name subnet-app \
  --subnet-prefixes 10.1.1.0/24

# Create Virtual Network 2
az network vnet create \
  --resource-group rg-networking-demo \
  --name vnet-analytics-east \
  --address-prefixes 10.2.0.0/16 \
  --subnet-name subnet-data \
  --subnet-prefixes 10.2.1.0/24

# Create VNet Peering from VNet-1 to VNet-2
az network vnet peering create \
  --name peering-east-to-analytics \
  --resource-group rg-networking-demo \
  --vnet-name vnet-production-east \
  --remote-vnet vnet-analytics-east \
  --allow-vnet-access

# Create return Peering from VNet-2 to VNet-1
az network vnet peering create \
  --name peering-analytics-to-east \
  --resource-group rg-networking-demo \
  --vnet-name vnet-analytics-east \
  --remote-vnet vnet-production-east \
  --allow-vnet-access
```

---

## 7. Summary Checklist
- [x] VNets provide isolated private IP boundaries in Azure.
- [x] Subnets partition a VNet into logical IP tiers; Azure reserves the first 4 and last 1 IP in every subnet.
- [x] Network Security Groups (NSGs) filter traffic using priority numbers (100–4096; lower = higher priority).
- [x] VNet Peering links disparate VNets over Microsoft's encrypted backbone with zero public internet exposure.
- [x] Peered VNets cannot have overlapping CIDR address spaces.
