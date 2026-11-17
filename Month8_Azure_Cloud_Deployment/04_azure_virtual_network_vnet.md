# 04 - Networking: Azure Virtual Network (VNet) & Peering

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `02:29:44` to `03:10:24`

---

## 1. What is an Azure Virtual Network (VNet)?

An **Azure Virtual Network (VNet)** is your own isolated, private network boundary inside the Microsoft Azure cloud. It is the fundamental foundational block for all private cloud networking.

In an on-premises enterprise office, servers and computers communicate with each other through physical routers, switches, and Ethernet cables. In Azure, a VNet provides that identical network topology virtually via software-defined networking (SDN):

```
+-------------------------------------------------------------------------+
|                       AZURE VIRTUAL NETWORK (VNet)                      |
|                           Address Space: 10.0.0.0/16                    |
+-------------------------------------------------------------------------+
|                                                                         |
|   +----------------------------+     +----------------------------+     |
|   |    Frontend Web Subnet     |     |   Backend Database Subnet  |     |
|   |        10.0.1.0/24         |     |        10.0.2.0/24         |     |
|   |                            |     |                            |     |
|   |   [ VM-Web-01 ]            |     |   [ VM-Database-01 ]       |     |
|   |   Private IP: 10.0.1.4     |     |   Private IP: 10.0.2.4     |     |
|   |   Public IP: 20.x.x.x      |     |   (No Public IP - Hidden!) |     |
|   +--------------+-------------+     +--------------^-------------+     |
|                  |                                  |                   |
|                  +======== Private VNet Traffic ====+                   |
|                            (Secure & Encrypted)                         |
+-------------------------------------------------------------------------+
```

### Key Communications Handled by a VNet:
1. **Communication among Azure Resources:** Virtual machines, AKS clusters, and Azure SQL Managed Instances communicate privately within the same VNet.
2. **Communication with the Public Internet:** Outbound internet access is enabled by default; inbound access is controlled strictly via Public IPs and Network Security Groups.
3. **Communication with On-Premises Networks:** Connect your corporate office or on-prem datacenter to Azure using **Azure VPN Gateway** (Site-to-Site IPsec) or **Azure ExpressRoute** (dedicated private fiber connection).

---

## 2. Understanding IP Addressing & CIDR Notation

### Public vs. Private IP Addresses
- **Private IP Address:** Routable exclusively *inside* your private VNet and peered networks. Devices on the public internet cannot see, ping, or connect to private IPs (e.g., `10.0.1.4`, `172.16.0.4`, `192.168.1.4`).
- **Public IP Address:** Globally unique and routable across the worldwide internet. Used when external users or customers access your web servers (e.g., `52.183.45.12`).

### RFC 1918 Private IP Address Spaces:
Azure VNets follow standard RFC 1918 private IPv4 allocation:
- `10.0.0.0` to `10.255.255.255` (Class A - 16,777,216 addresses - Most common for enterprise clouds)
- `172.16.0.0` to `172.31.255.255` (Class B - 1,048,576 addresses)
- `192.168.0.0` to `192.168.255.255` (Class C - 65,536 addresses)

---

### Demystifying CIDR Notation (Classless Inter-Domain Routing)

In CIDR notation, the slash number (`/X`) indicates how many of the 32 bits are locked for the **Network ID**, leaving the remaining `(32 - X)` bits for **Host IDs**:

```
CIDR Calculation: 32 bits total
Example: 10.0.0.0/16
- Locked Network Bits: 16 bits (10.0)
- Host Bits: 16 bits (2^16 = 65,536 total IP addresses)

Example: 10.0.1.0/24
- Locked Network Bits: 24 bits (10.0.1)
- Host Bits: 8 bits (2^8 = 256 total IP addresses)
```

| CIDR Prefix | Total IP Addresses | Azure Usable IPs | Common Use Case |
| :--- | :--- | :--- | :--- |
| **/16** (e.g., `10.0.0.0/16`) | 65,536 | 65,531 | Standard Virtual Network overall address space |
| **/24** (e.g., `10.0.1.0/24`) | 256 | 251 | Standard Subnet for Web, App, or DB tiers |
| **/26** (e.g., `10.0.3.0/26`) | 64 | 59 | Azure Bastion or Application Gateway Subnet |
| **/27** (e.g., `10.0.4.0/27`) | 32 | 27 | Azure Firewall or VPN Gateway Subnet |
| **/28** (e.g., `10.0.5.0/28`) | 16 | 11 | Small specialized utility subnet |

---

### The 5 Azure Reserved IP Addresses (Critical Concept!)

In any standard networking environment, 2 IPs are reserved (Network and Broadcast). However, in **every subnet created in Microsoft Azure, exactly 5 IP addresses are reserved**:

```
Suppose your Subnet is: 10.0.1.0/24 (256 Total IPs)
---------------------------------------------------------------------------
1. 10.0.1.0   -> Network Address (Protocol standard)
2. 10.0.1.1   -> Default Gateway (Azure routes traffic out of the subnet)
3. 10.0.1.2   -> Azure DNS Mapping (Resolves Azure hostnames)
4. 10.0.1.3   -> Azure DNS Secondary / Internal Fabric Coordination
5. 10.0.1.255 -> Network Broadcast Address (Azure does not use broadcast)
---------------------------------------------------------------------------
Result: Available usable IPs = 256 - 5 = 251 IPs.
The very FIRST virtual machine deployed will ALWAYS receive: 10.0.1.4!
```

---

## 3. Subnets & Multi-Tier Architecture

A **Subnet** is a range of IP addresses within a VNet. By carving a VNet into logical subnets, you can establish an enterprise multi-tier architecture with defense-in-depth security:

```
+---------------------------------------------------------------------------+
|                   ENTERPRISE 3-TIER SUBNET ARCHITECTURE                   |
+---------------------------------------------------------------------------+
|                                                                           |
|  1. PUBLIC WEB SUBNET (10.0.1.0/24):                                      |
|     - Hosts: Public Load Balancer & Nginx Ingress Controllers.            |
|     - NSG Rule: Allows Inbound Port 80/443 from Internet.                 |
|                                |                                          |
|                                v Internal private traffic only            |
|  2. PRIVATE APPLICATION SUBNET (10.0.2.0/24):                             |
|     - Hosts: Backend microservices, Node.js / Java business logic.        |
|     - NSG Rule: Allows traffic ONLY from Web Subnet; drops Internet!      |
|                                |                                          |
|                                v Internal private traffic only            |
|  3. ISOLATED DATABASE SUBNET (10.0.3.0/24):                               |
|     - Hosts: SQL Server, PostgreSQL, MySQL databases.                     |
|     - NSG Rule: Allows Port 1433/5432 ONLY from Application Subnet!       |
|     - Completely inaccessible from the internet or web tier!              |
|                                                                           |
+---------------------------------------------------------------------------+
```

---

## 4. Network Security Groups (NSGs): The Cloud Firewall

A **Network Security Group (NSG)** acts as a stateful packet-filtering firewall that controls inbound and outbound network traffic to network interfaces (NICs) or subnets.

### The 5-Tuple Rule Structure:
Every rule in an NSG is evaluated using 5 specific attributes:
1. **Source:** Where the traffic comes from (IP address, CIDR block, or Service Tag like `Internet`).
2. **Source Port Range:** The port used by the client (usually `*` / ephemeral).
3. **Destination:** Where the traffic is going (IP address, CIDR block, or Subnet).
4. **Destination Port Range:** The port the service listens on (e.g., `80`, `443`, `22`).
5. **Protocol:** `TCP`, `UDP`, or `Any`.

```
Incoming Internet Packet: Source=Any, Destination Port=80, Protocol=TCP
                                  |
                                  v
+-------------------------------------------------------------------------+
|                  NSG RULE EVALUATION FLOWCHART                          |
+-------------------------------------------------------------------------+
|                                                                         |
|  Rule Priority 100: Allow SSH (Port 22)       -> Match? NO -> Next      |
|                                                                         |
|  Rule Priority 200: Allow HTTP (Port 80)      -> Match? YES!            |
|  ===> ACTION: ALLOW! Traffic admitted! Evaluation TERMINATES!           |
|                                                                         |
|  Rule Priority 300: Deny All Web              -> SKIPPED!               |
|                                                                         |
|  Rule Priority 65000: Default AllowVNet       -> SKIPPED!               |
|  Rule Priority 65500: Default DenyAllInBound  -> SKIPPED!               |
+-------------------------------------------------------------------------+
```

### The 3 Golden Rules of NSGs:
1. **Priority Values (100 to 4096):** Lower numbers have higher priority and are evaluated first!
2. **First Match Wins:** As soon as an incoming packet matches a rule, processing stops immediately.
3. **NSGs are Stateful:** If you allow inbound traffic on port 80, the outgoing return response packet is **automatically allowed** regardless of outbound rules!

### Built-in Default Rules (Cannot be deleted):
- `AllowVNetInBound` (Priority 65000): Allows all resources within the same VNet to talk to each other.
- `AllowAzureLoadBalancerInBound` (Priority 65001): Allows Azure health probes to check server status.
- `DenyAllInBound` (Priority 65500): Drops all other incoming traffic from the outside world.

---

## 5. What is Azure VNet Peering?

By default, Virtual Networks are completely isolated from one another. A VM in `VNet-A` cannot communicate with a VM in `VNet-B`, even if they exist in the exact same Azure subscription and region!

**Azure VNet Peering** connects two Virtual Networks seamlessly:
- Traffic routes directly over **Microsoft’s private fiber backbone**, never touching the public internet.
- Delivers the same ultra-low latency and high bandwidth as if both VMs were in the exact same network.
- **Two Varieties:**
  1. **Regional VNet Peering:** Connects VNets in the **same** Azure region (e.g., `East US` to `East US`).
  2. **Global VNet Peering:** Connects VNets in **different** Azure regions worldwide (e.g., `East US` to `West Europe`).

```
+--------------------------+                      +--------------------------+
|      VNet-1 (East US)    |                      |    VNet-2 (West Europe)  |
|   Address: 10.1.0.0/16   |                      |   Address: 10.2.0.0/16   |
|                          |                      |                          |
|    [ VM-East-01 ]        |                      |    [ VM-Europe-01 ]      |
|    IP: 10.1.1.4          |                      |    IP: 10.2.1.4          |
+------------+-------------+                      +-------------^------------+
             |                                                  |
             +======== Azure Global VNet Peering ===============+
                       - 100% Private Microsoft Backbone
                       - Zero Public Internet Exposure
                       - Sub-millisecond regional latency
```

> [!CAUTION]
> **Non-Overlapping CIDR Rule:** VNet Peering **strictly prohibits** overlapping IP address spaces. If VNet-1 is `10.0.0.0/16`, VNet-2 cannot be `10.0.0.0/16`. They must have distinct CIDR blocks (e.g., `10.1.0.0/16` and `10.2.0.0/16`).

---

## 6. Hands-on Lab: Deploying Two VNets & Configuring Peering

In this lab from the video, M Prashant proves that two separate VNets cannot communicate until bidirectional Peering is established:

### Step 1: Create Resource Group
```bash
az group create --name rg-networking-demo --location eastus
```

---

### Step 2: Create VNet-1 and Deploy VM-1
1. **Create VNet-1:**
   - Name: `vnet-production-east`
   - Address Space: `10.1.0.0/16`
   - Subnet Name: `subnet-app`
   - Subnet Prefix: `10.1.1.0/24`
2. **Deploy VM-1:**
   - Name: `vm-east-01`
   - Image: `Ubuntu 22.04 LTS`
   - Subnet: `subnet-app` in `vnet-production-east`
   - Authentication: SSH key or password
   - Note Private IP: `10.1.1.4`

---

### Step 3: Create VNet-2 and Deploy VM-2
1. **Create VNet-2:**
   - Name: `vnet-analytics-east`
   - Address Space: `10.2.0.0/16` *(Notice distinct, non-overlapping range!)*
   - Subnet Name: `subnet-data`
   - Subnet Prefix: `10.2.1.0/24`
2. **Deploy VM-2:**
   - Name: `vm-analytics-01`
   - Image: `Ubuntu 22.04 LTS`
   - Subnet: `subnet-data` in `vnet-analytics-east`
   - Note Private IP: `10.2.1.4`

---

### Step 4: Test Pre-Peering Connectivity (Expected Failure)
1. SSH into `vm-east-01` using its Public IP.
2. Attempt to ping the private IP of `vm-analytics-01`:
   ```bash
   ping -c 4 10.2.1.4
   ```
3. **Result:** `100% packet loss` (Request timed out). The two networks are completely isolated.

---

### Step 5: Configure Bidirectional VNet Peering in the Portal
1. In the Azure Portal, open `vnet-production-east`.
2. Under **Settings** in the left menu, click **Peerings**.
3. Click **+ Add**:
   - **This virtual network (vnet-production-east):**
     - Peering link name: `peer-east-to-analytics`
     - Traffic to remote virtual network: **Allow (default)**
     - Traffic forwarded from remote virtual network: **Allow (default)**
   - **Remote virtual network (vnet-analytics-east):**
     - Peering link name: `peer-analytics-to-east`
     - Virtual network deployment model: `Resource manager`
     - Select: `vnet-analytics-east`
     - Traffic to remote virtual network: **Allow (default)**
4. Click **Add**. Azure provisions both peering links simultaneously.
5. In the Peering table, verify that the **Peering status** reads: **Connected**.

---

### Step 6: Verify Post-Peering Connectivity (Success!)
1. Return to your terminal session on `vm-east-01`.
2. Re-run the ping command:
   ```bash
   ping -c 4 10.2.1.4
   ```
3. **Result:** Success! Packets are answered immediately with sub-millisecond response times:
   ```
   64 bytes from 10.2.1.4: icmp_seq=1 ttl=64 time=0.842 ms
   64 bytes from 10.2.1.4: icmp_seq=2 ttl=64 time=0.761 ms
   64 bytes from 10.2.1.4: icmp_seq=3 ttl=64 time=0.755 ms
   64 bytes from 10.2.1.4: icmp_seq=4 ttl=64 time=0.749 ms

   --- 10.2.1.4 ping statistics ---
   4 packets transmitted, 4 received, 0% packet loss, time 3004ms
   ```

---

## 7. Troubleshooting Common Azure Networking Issues

| Issue | Likely Cause | Solution |
| :--- | :--- | :--- |
| Ping between peered VMs fails despite Peering showing "Connected". | Ubuntu/Linux firewall (UFW) or Azure NSG is blocking ICMP protocol. | Add an NSG inbound rule allowing protocol `ICMP` from source CIDR `10.0.0.0/8`. Ensure OS firewall `ufw allow proto icmp` is enabled. |
| Cannot create VNet Peering link. | The two VNets have overlapping IP address spaces. | You cannot peer overlapping CIDRs. You must recreate one of the VNets with a non-overlapping address block. |
| VNet Peering status says "Initiated" instead of "Connected". | Only one direction of the peering link was created. | VNet peering must be created in both directions (VNet1 -> VNet2 AND VNet2 -> VNet1). |
| Virtual machine does not receive the expected private IP. | The first 4 IP addresses (.0 to .3) are reserved by Azure. | The first VM in `10.0.1.0/24` will always receive `10.0.1.4`. |

---

## 8. Summary Checklist
- [x] Azure Virtual Networks provide secure, isolated private IP environments for cloud resources.
- [x] Subnets partition a VNet into tiers; Azure reserves 5 IP addresses in every subnet (.0, .1, .2, .3, .255).
- [x] Network Security Groups (NSGs) act as stateful firewalls evaluated by priority (100–4096; lowest number wins).
- [x] VNet Peering links separate VNets over Microsoft's encrypted backbone with zero public internet traversal.
- [x] Peered VNets must have strictly non-overlapping CIDR address blocks.
