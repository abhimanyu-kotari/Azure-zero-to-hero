# 04 - Azure Virtual Network (VNet) & Networking

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `02:29:44` to `03:10:24`

---

## 1. What is an Azure Virtual Network (VNet)?

An **Azure Virtual Network (VNet)** is your own private, isolated digital network in the Microsoft cloud. 

Just like your home Wi-Fi router connects your laptop, phone, and smart TV into a private network where they can talk to each other safely, an Azure VNet connects your Virtual Machines, databases, and container instances together securely.

### Key Capabilities of a VNet:
- **Isolation:** Your VNet is strictly isolated from other customers' VNets in Azure.
- **Internal Communication:** Resources inside the same VNet communicate over private IP addresses with high speed and zero data transfer fees within the same zone.
- **Internet Communication:** By default, all resources in a VNet can make outbound calls to the internet. Inbound traffic from the internet is blocked unless assigned a Public IP or routed via a Load Balancer.
- **Hybrid Cloud:** You can securely connect your on-premises office network to your Azure VNet using an encrypted VPN Gateway or dedicated ExpressRoute.

---

## 2. Understanding IP Addresses & CIDR Notation

To build a network, you must assign it an address range using **CIDR (Classless Inter-Domain Routing)** notation.

### What Does `10.0.0.0/16` Mean?
An IPv4 address consists of 4 numbers separated by dots (32 bits in total). The `/` number tells you how many bits are locked for the network address, and how many are free for your devices:

```
Address: 10 . 0 . 0 . 0 / 16
         [ Locked Network ] [ Free for Hosts ]
```

- **/16 Subnet:** Gives you **65,536 total IP addresses** (ranging from `10.0.0.0` to `10.0.255.255`). This is ideal for your main Virtual Network.
- **/24 Subnet:** Gives you **256 total IP addresses** (ranging from `10.0.1.0` to `10.0.1.255`). This is standard for individual subnets.

### The 5 Reserved IPs in Every Azure Subnet
In standard networking, 2 IP addresses are reserved (Network and Broadcast). **In Azure, Microsoft reserves 5 IP addresses in every single subnet:**
If your subnet is `10.0.1.0/24` (256 theoretical IPs):
1. `10.0.1.0`: Network address.
2. `10.0.1.1`: Default gateway (used by Azure to route traffic).
3. `10.0.1.2`: Azure DNS service mapping.
4. `10.0.1.3`: Azure DNS service mapping.
5. `10.0.1.255`: Network broadcast address.
- **Available for your VMs:** $256 - 5 = \mathbf{251\text{ usable IP addresses}}$.

---

## 3. Subnets: Organizing Your Network into Tiers

You should never place all your servers into a single flat network. Instead, you divide your VNet into smaller logical segments called **Subnets**:

```
+-------------------------------------------------------------------------------+
|                       VIRTUAL NETWORK: 10.0.0.0/16                            |
|                                                                               |
|  +--------------------------------+   +------------------------------------+  |
|  |     WEB SUBNET: 10.0.1.0/24    |   |     DATABASE SUBNET: 10.0.2.0/24   |  |
|  |     (Public Facing Tier)       |   |     (Strictly Private Tier)        |  |
|  |                                |   |                                    |  |
|  |   [ Frontend Web VM: .4 ]      |   |   [ Backend Database VM: .4 ]      |  |
|  |   - Has Public IP              |   |   - NO Public IP                   |  |
|  |   - Open to HTTP (Port 80)     |   |   - Accessible ONLY from Web Subnet|  |
|  +--------------------------------+   +------------------------------------+  |
+-------------------------------------------------------------------------------+
```

### Best Practice Multi-Tier Design:
1. **Web Subnet (`10.0.1.0/24`):** Holds load balancers and frontend web servers that internet visitors can reach.
2. **Backend API Subnet (`10.0.2.0/24`):** Holds microservices and application logic. Not accessible from the internet.
3. **Database Subnet (`10.0.3.0/24`):** Holds SQL databases or MongoDB. Has zero public IPs; accepts connections only from the API subnet.

---

## 4. Network Security Groups (NSGs): Your Cloud Firewall

A **Network Security Group (NSG)** is a virtual firewall that filters network traffic arriving at or leaving your Azure resources.

### Key Rules of NSGs:
1. **Inbound vs. Outbound:**
   - **Inbound Rules:** Control traffic coming *into* your subnet or VM (e.g., Allow port 80 HTTP, Deny port 22 SSH from public).
   - **Outbound Rules:** Control traffic going *out* from your VM to other servers or the internet.
2. **Stateful Filtering:**
   - NSGs are **stateful**. If you create an Inbound rule allowing a user to connect on Port 80, the return response from the server is **automatically allowed** out. You do not need to create a matching outbound rule!
3. **Priority Numbers (100 to 4096):**
   - Rules are evaluated in strict numerical order from **lowest to highest**.
   - If Rule 100 says `Allow Port 80` and Rule 200 says `Deny Port 80`, Azure applies Rule 100 and stops checking.
   - Lowest number has the highest priority!
4. **Association Level:**
   - **Subnet Level (Recommended):** Protects all VMs inside that entire subnet with a single set of rules.
   - **NIC Level:** Protects only a single specific virtual machine's network card.

---

## 5. VNet Peering: Connecting Two Networks Privately

What if your company has two separate VNets—for example, `Marketing-VNet` and `Finance-VNet`—and servers in both need to share data?

**VNet Peering** connects two Azure Virtual Networks seamlessly:
- Traffic between peered VNets travels entirely across **Microsoft's private fiber-optic backbone network**.
- Traffic **never touches the public internet**.
- Latency is ultra-low, bandwidth is high, and data stays encrypted within Microsoft infrastructure.
- **Global VNet Peering:** Connects VNets located in completely different continents (e.g., East US to West Europe) with private routing!

---

## 6. Hands-on Walkthrough: Creating a Custom VNet & Subnets

In the video, M Prashant builds a secure multi-subnet network:

### Step 1: Create the Virtual Network
1. Search for **Virtual networks** in the Azure Portal -> Click **Create**.
2. Resource Group: `rg-networking-demo`.
3. Name: `prod-vnet`.
4. Region: `East US`.
5. Under **IP addresses**:
   - IPv4 address space: `10.0.0.0/16`.
6. Click **+ Add subnet**:
   - Subnet 1 Name: `web-subnet` -> Range: `10.0.1.0/24`.
7. Click **+ Add subnet**:
   - Subnet 2 Name: `db-subnet` -> Range: `10.0.2.0/24`.
8. Click **Review + create** -> **Create**.

### Step 2: Create and Attach an NSG
1. Search for **Network security groups** -> Click **Create**.
2. Name: `nsg-web-tier`.
3. Go to the new NSG -> Click **Inbound security rules** -> **+ Add**:
   - Destination port ranges: `80`
   - Protocol: `TCP`, Action: `Allow`, Priority: `100`, Name: `Allow-HTTP`.
4. Click **Subnets** on the left menu -> Click **+ Associate**:
   - Select `prod-vnet`.
   - Select `web-subnet`.
   - Click **OK**.

Now, any virtual machine you deploy inside `web-subnet` is automatically shielded by this firewall!

---

## 7. Summary Checklist
- [x] VNets provide isolated, private networking for your cloud resources.
- [x] Subnets break a large network into manageable, secure tiers (Web, App, DB).
- [x] Remember: Azure reserves 5 IP addresses in every subnet.
- [x] Network Security Groups (NSGs) filter traffic using priority-ordered, stateful rules.
- [x] VNet Peering links two virtual networks privately over Microsoft's high-speed backbone.
