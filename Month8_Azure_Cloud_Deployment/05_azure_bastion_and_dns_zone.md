# 05 - Security & Routing: Azure Bastion & Azure DNS Zones

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `03:10:24` to `03:36:13`

---

## 1. The Security Danger of Exposing Management Ports

In traditional hosting and first-time cloud setups, administrators assign a Public IP address to their virtual machines and open management ports:
- **Port 22:** Secure Shell (SSH) for Linux servers.
- **Port 3389:** Remote Desktop Protocol (RDP) for Windows servers.

### Why is this an extreme security risk?
1. **Automated Port Scanners:** Millions of automated botnets continuously scan the entire IPv4 address space searching for open ports 22 and 3389.
2. **Brute-Force & Credential Stuffing:** Once detected, bots execute dictionary attacks testing hundreds of thousands of username/password combinations per hour.
3. **Zero-Day Vulnerabilities:** Any unpatched exploit in the OpenSSH or Windows RDP protocol can grant an attacker full root or SYSTEM level access to your cloud environment.

Historically, organizations set up a physical "Jump Box" or "Bastion Server". However, managing a standalone jump host required maintaining an additional VM, patching its operating system, configuring firewall rules, and handling scaling.

---

## 2. What is Azure Bastion?

**Azure Bastion** is a fully managed Platform as a Service (PaaS) that provides secure, seamless RDP and SSH connectivity to your virtual machines directly through the **Azure Portal over TLS/HTTPS (Port 443)**.

```
+-------------------------------------------------------------------------+
|                        AZURE BASTION ARCHITECTURE                       |
+-------------------------------------------------------------------------+
|                                                                         |
|  Administrator Laptop / Web Browser                                     |
|       |                                                                 |
|       | HTTPS / TLS (Port 443 only - standard encrypted web traffic)    |
|       v                                                                 |
|  +-------------------------------------------------------------------+  |
|  | Azure Virtual Network (VNet)                                      |  |
|  |                                                                   |  |
|  |  +-------------------------------------------------------------+  |  |
|  |  | AzureBastionSubnet (Requires /26 or larger)                 |  |  |
|  |  |   [ AZURE BASTION PaaS SERVICE ]                            |  |  |
|  |  +------------------------------+------------------------------+  |  |
|  |                                 |                                 |  |
|  |                                 | Private IP (Internal SSH/RDP)   |  |
|  |                                 v                                 |  |
|  |  +-------------------------------------------------------------+  |  |
|  |  | Private Workload Subnet (10.0.1.0/24)                        |  |  |
|  |  |   [ VM-Private-01 ] (Private IP: 10.0.1.4)                  |  |  |
|  |  |   * NO PUBLIC IP ASSIGNED!                                  |  |  |
|  |  |   * PORTS 22 & 3389 BLOCKED FROM THE PUBLIC INTERNET!       |  |  |
|  |  +-------------------------------------------------------------+  |  |
|  +-------------------------------------------------------------------+  |
+-------------------------------------------------------------------------+
```

### Core Security Advantages of Azure Bastion:
- **Zero Public IPs on VMs:** Your virtual machines live purely in private subnets with private IPs.
- **Browser-Based Access:** Connect via HTML5 directly in any modern browser (Chrome, Edge, Firefox, Safari). No PuTTY, OpenSSH client, or Remote Desktop client required on your computer!
- **No Inbound Port Holes:** You do **not** need to open ports 22 or 3389 on your VM’s Network Security Group.
- **Complete Protection from Scanners:** Because external communication terminates at Azure Bastion via HTTPS (Port 443), your virtual machines are completely invisible to public internet port scanners.

---

## 3. Hands-on Lab: Deploying and Connecting with Azure Bastion

### Step 1: Create the Dedicated Bastion Subnet
Azure Bastion enforces strict networking requirements:
1. The subnet name **MUST be named exactly**: `AzureBastionSubnet` (case-sensitive).
2. The subnet prefix must be **/26 or larger** (e.g., `/26`, `/25`, `/24`) to give Bastion sufficient IP space for automated host scaling.

1. Open your Virtual Network (`vnet-production-east`) in the Azure Portal.
2. Under **Settings** in the left menu, click **Subnets** -> **+ Subnet**.
3. **Name:** `AzureBastionSubnet`
4. **Starting IP address:** `10.1.2.0/26` (or any available non-overlapping range).
5. Click **Save**.

---

### Step 2: Deploy Azure Bastion
1. In the top search bar, search for **Bastions** and click **+ Create**.
2. **Basics Tab:**
   - **Subscription:** Select your subscription.
   - **Resource group:** `rg-networking-demo`.
   - **Name:** `bastion-secure-proxy`.
   - **Region:** `East US` (Must match your VNet region).
   - **Tier:** **Basic** (Cost-effective, supports browser SSH/RDP) or **Standard** (Adds native client support and host scaling).
   - **Virtual network:** Select `vnet-production-east`.
   - **Subnet:** Automatically selects `AzureBastionSubnet`.
   - **Public IP address:** Create new -> Name: `pip-bastion-proxy`.
3. Click **Review + create** -> **Create**.
4. Azure takes approximately 5–7 minutes to provision the managed Bastion instances.

---

### Step 3: Connect to a Private VM via Bastion
1. Navigate to your private virtual machine (`vm-east-01` which has **no** public IP).
2. At the top of the VM overview page, click **Connect** -> **Bastion**.
3. Fill in the connection parameters:
   - **Authentication Type:** `SSH Private Key from Local File` or `Password`.
   - **Username:** `azureuser`
   - **SSH Private Key:** Browse and upload your downloaded `.pem` private key.
4. Click **Connect**.
5. A new browser tab opens immediately with a full-screen Linux terminal session running smoothly inside your web browser over secure HTTPS!

---

## 4. What is Azure DNS & DNS Zones?

The **Domain Name System (DNS)** is the global directory of the internet. It translates human-friendly domain names (e.g., `www.mycoolcloudapp.com`) into computer-routable numerical IP addresses (e.g., `20.120.45.67`).

```
User types "www.mycoolcloudapp.com" in browser
                       |
                       v
       +-------------------------------+
       |   Azure DNS Global Anycast    |
       |         Nameservers           |
       +---------------+---------------+
                       |
                       | Resolves to: 20.120.45.67
                       v
Browser connects directly to Azure Load Balancer or VM Public IP!
```

### Public vs. Private DNS Zones

| Feature | Azure Public DNS Zone | Azure Private DNS Zone |
| :--- | :--- | :--- |
| **Scope & Visibility** | Globally resolvable by anyone on the public internet. | Resolvable **only** by VMs inside linked Azure Virtual Networks. |
| **Typical Use Case** | Routing customer traffic to public web apps, Front Door, or APIs. | Internal microservice discovery (e.g., `api.internal.corp`, `db.production.local`). |
| **Domain Registration** | Requires purchasing a domain name from a registrar (GoDaddy, Namecheap). | Does not require purchasing a domain; you can invent any private naming convention! |
| **Split-Horizon DNS** | Directs external internet users to public IPs. | Directs internal VMs to private IPs using the exact same domain name. |

---

## 5. Common DNS Record Types Cheat Sheet

When managing a DNS Zone, you create **Record Sets**:

| Record Type | Full Name | Purpose | Real-World Example |
| :--- | :--- | :--- | :--- |
| **A Record** | Address | Maps a hostname to a 32-bit **IPv4 address**. | `www` -> `20.120.45.67` |
| **AAAA Record** | IPv6 Address | Maps a hostname to a 128-bit **IPv6 address**. | `www` -> `2001:0db8::1` |
| **CNAME** | Canonical Name | Maps an alias name to another true domain name. | `blog` -> `mycompany.azurewebsites.net` |
| **MX Record** | Mail Exchanger | Directs incoming company emails to mail servers. | `10 mail.protection.outlook.com` |
| **TXT Record** | Text | Stores text for domain verification & anti-spam (SPF/DKIM). | `v=spf1 include:spf.protection.outlook.com -all` |
| **NS Record** | Name Server | Delegates a DNS zone to use specific authoritative nameservers. | `ns1-01.azure-dns.com` |

---

## 6. Hands-on Lab: Configuring an Azure Public DNS Zone

### Step 1: Create the DNS Zone in Azure
1. In the Azure Portal search bar, search for **DNS zones** and click **+ Create**.
2. **Basics Tab:**
   - **Subscription:** Select your subscription.
   - **Resource group:** `rg-networking-demo`.
   - **Name:** Enter your registered domain name (e.g., `learnazurewithprashant.com`).
   - Click **Review + create** -> **Create**.

---

### Step 2: Note Azure's 4 Authoritative Name Servers
Open your newly created DNS Zone. On the **Overview** page, Azure displays **4 authoritative Name Servers**:
```
ns1-01.azure-dns.com
ns2-01.azure-dns.net
ns3-01.azure-dns.org
ns4-01.azure-dns.info
```

### Step 3: Delegate the Domain at Your Registrar
1. Log into your domain registrar where you purchased the domain (e.g., GoDaddy, Namecheap, Cloudflare, Google Domains).
2. Locate the **DNS Management / Nameservers** section.
3. Select **Custom Nameservers** and replace the default nameservers with Azure's 4 nameservers.
4. Save changes. *(DNS propagation begins immediately and can take from 5 minutes up to 24 hours).*

---

### Step 4: Create an A Record Pointing to Your Azure Server
1. In your Azure DNS Zone, click **+ Record set**.
2. **Name:** `www`
3. **Type:** `A - IPv4 address`.
4. **TTL (Time to Live):** `3600` (1 hour).
5. **IP address:** Enter the Public IP of your Azure Load Balancer or VM (e.g., `20.120.45.67`).
6. Click **OK**.

### Step 5: Test DNS Resolution from Terminal
Open your local computer terminal or PowerShell and verify DNS resolution:
```bash
nslookup www.learnazurewithprashant.com
```
**Expected Output:**
```
Server:  dns.google
Address: 8.8.8.8

Non-authoritative answer:
Name:    www.learnazurewithprashant.com
Address: 20.120.45.67
```

---

## 7. Troubleshooting Bastion & DNS

| Symptom | Cause | Resolution |
| :--- | :--- | :--- |
| Cannot create Bastion; subnet selector is disabled. | Missing or incorrectly named subnet. | Subnet name must be typed exactly as `AzureBastionSubnet` with prefix `/26` or larger. |
| Bastion browser tab opens but says "Connection failed". | Target VM is powered off or VM NSG blocks Bastion. | Verify target VM is in the `Running` state. Ensure the VM's subnet NSG permits traffic from `AzureBastionSubnet`. |
| Domain name does not resolve after adding A record. | DNS TTL caching or registrar delegation pending. | Test directly against Azure's nameservers: `nslookup www.yourdomain.com ns1-01.azure-dns.com`. If that works, wait for global ISP cache TTL to expire. |

---

## 8. Summary Checklist
- [x] Leaving ports 22 and 3389 exposed to the public internet attracts automated brute-force attacks.
- [x] Azure Bastion delivers secure RDP/SSH browser-based management over HTTPS (Port 443) with zero public IPs on VMs.
- [x] Bastion requires a dedicated subnet named strictly `AzureBastionSubnet` with `/26` or larger prefix.
- [x] Azure Public DNS hosts domain records on Microsoft's global high-availability Anycast network.
- [x] Private DNS zones enable internal name resolution for microservices without public internet exposure.
