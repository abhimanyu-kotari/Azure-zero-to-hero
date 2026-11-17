# 05 - Security & Routing: Azure Bastion & Azure DNS Zones

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `03:10:24` to `03:36:13`

---

## 1. The Security Danger of Public IPs on Servers

In traditional setups, engineers attach a Public IP address to their virtual machines and open port `22` (SSH for Linux) or port `3389` (RDP for Windows) so administrators can log in.

### Why is this dangerous?
- **Port Scanners & Bots:** Within minutes of assigning a public IP, automated bots scan the internet for open ports 22 and 3389.
- **Brute-Force Attacks:** Hackers launch dictionary attacks trying thousands of username and password combinations.
- **Zero-Day Exploits:** Any unpatched vulnerability in the SSH or RDP daemon can compromise the entire server.

Historically, organizations set up a physical "Jump Server" or "Bastion Host". However, managing the jump host itself (OS patching, hardening, scaling) required ongoing maintenance.

---

## 2. What is Azure Bastion?

**Azure Bastion** is a fully managed Platform as a Service (PaaS) that provides secure, seamless RDP and SSH connectivity to your virtual machines directly through the **Azure Portal over SSL (Port 443)**.

```
+-------------------------------------------------------------------------+
|                        AZURE BASTION ARCHITECTURE                       |
+-------------------------------------------------------------------------+
|                                                                         |
|  Administrator Laptop                                                   |
|       |                                                                 |
|       | HTTPS / TLS (Port 443 only!)                                    |
|       v                                                                 |
|  +-------------------------------------------------------------------+  |
|  | Azure Virtual Network (VNet)                                      |  |
|  |                                                                   |  |
|  |  +-------------------------------------------------------------+  |  |
|  |  | AzureBastionSubnet (Requires /26 or larger)                 |  |  |
|  |  |   [ AZURE BASTION PaaS SERVICE ]                            |  |  |
|  |  +------------------------------+------------------------------+  |  |
|  |                                 |                                 |  |
|  |                                 | Private IP (RDP / SSH)          |  |
|  |                                 v                                 |  |
|  |  +-------------------------------------------------------------+  |  |
|  |  | Workload Subnet (10.0.1.0/24)                                |  |  |
|  |  |   [ VM-Private-01 ] (Private IP: 10.0.1.4, NO PUBLIC IP!)   |  |  |
|  |  +-------------------------------------------------------------+  |  |
|  +-------------------------------------------------------------------+  |
+-------------------------------------------------------------------------+
```

### Core Security Advantages of Azure Bastion:
1. **Zero Public IPs on VMs:** Your virtual machines live purely in private subnets with private IPs.
2. **Browser-Based Access:** Connect via HTML5 in any modern web browser (Edge, Chrome, Safari). No SSH client, PuTTY, or RDP client software required.
3. **No Port Openings:** You do not need to allow inbound ports 22 or 3389 on your VM's Network Security Group.
4. **Protection Against Scanners:** Because traffic terminates at Azure Bastion via HTTPS (port 443), your VM is completely invisible to public internet port scanners.

---

## 3. Hands-on Lab: Deploying and Using Azure Bastion

### Step 1: Create the Dedicated Bastion Subnet
Azure Bastion has strict networking prerequisites:
- The subnet name **MUST be exactly** `AzureBastionSubnet` (case-sensitive).
- The subnet prefix must be **/26 or larger** (e.g., `/26`, `/25`, `/24`) to accommodate Bastion scaling.

1. Go to your Virtual Network in the Azure Portal.
2. Under **Settings**, click **Subnets** -> **+ Subnet**.
3. **Name:** `AzureBastionSubnet`
4. **Starting IP address:** `10.0.3.0/26` (or any available non-overlapping range).
5. Click **Save**.

---

### Step 2: Deploy Azure Bastion
1. Search for **Bastions** in the Azure Portal search bar.
2. Click **+ Create**.
3. **Basics Tab:**
   - **Subscription:** Select your subscription.
   - **Resource Group:** `rg-networking-demo`
   - **Name:** `bastion-secure-access`
   - **Region:** Same region as your VNet (e.g., `East US`).
   - **Tier:** **Basic** (cost-effective, supports browser SSH/RDP) or **Standard** (adds host scaling & native client support).
   - **Virtual Network:** Select your VNet.
   - **Subnet:** It will automatically lock onto `AzureBastionSubnet`.
   - **Public IP address:** Create new -> `pip-bastion-east`.
4. Click **Review + create** -> **Create**.
5. Deployment takes approximately 5–7 minutes.

---

### Step 3: Connect to a Private VM using Bastion
1. Navigate to your virtual machine (which has **no** public IP).
2. At the top of the VM page, click **Connect** -> **Bastion**.
3. Enter your login credentials:
   - **Authentication Type:** `SSH Private Key from Local File` or `Password`.
   - **Username:** `azureuser`
   - Select your `.pem` key file.
4. Click **Connect**.
5. A new browser tab opens immediately with a full Linux terminal session running directly inside your browser window over secure HTTPS!

---

## 4. What is Azure DNS & DNS Zones?

The **Domain Name System (DNS)** is the address book of the internet. It translates human-friendly domain names (e.g., `www.mycoolapp.com`) into computer-routable IP addresses (e.g., `20.120.45.67`).

```
User types "mycoolapp.com"
            |
            v
+-----------------------+
|  Azure DNS Nameserver |  ---- "mycoolapp.com is at 20.120.45.67"
+-----------------------+
            |
            v
Browser connects directly to Azure Public IP 20.120.45.67!
```

### Public vs. Private DNS Zones

| Feature | Azure Public DNS Zone | Azure Private DNS Zone |
| :--- | :--- | :--- |
| **Visibility** | Globally accessible across the public internet. | Accessible **only** within designated VNets. |
| **Use Case** | Routing customer traffic to public load balancers or VMs. | Internal microservice resolution (e.g., `db.internal.corp`). |
| **Domain Ownership** | Requires a registered domain (from GoDaddy, Namecheap, etc.). | Can use custom fictitious domains (e.g., `.local`, `.corp`). |
| **Split-Horizon** | Resolves external clients. | Resolves internal VMs without exposing IPs externally. |

---

## 5. Common DNS Record Types

When configuring an Azure DNS Zone, you create **Record Sets**:

| Record Type | Name | Purpose | Example |
| :--- | :--- | :--- | :--- |
| **A Record** | Address | Maps a hostname to an **IPv4 address**. | `@` -> `20.120.45.67` |
| **AAAA Record** | IPv6 Address | Maps a hostname to an **IPv6 address**. | `@` -> `2001:0db8::1` |
| **CNAME** | Canonical Name | Maps an alias hostname to another domain name. | `www` -> `mycoolapp.azurewebsites.net` |
| **MX Record** | Mail Exchanger | Directs incoming emails to mail servers. | `10 mail.google.com` |
| **TXT Record** | Text | Stores text for domain verification & anti-spam (SPF/DKIM). | `v=spf1 include:_spf.google.com ~all` |

---

## 6. Hands-on Lab: Configuring an Azure Public DNS Zone

### Step 1: Create the DNS Zone in Azure
1. In the Azure Portal, search for **DNS zones** and click **+ Create**.
2. **Resource group:** `rg-networking-demo`
3. **Name:** Enter your purchased domain name (e.g., `learnazurewithprashant.com`).
4. Click **Review + create** -> **Create**.

---

### Step 2: Note the Azure Name Servers (NS Records)
Once created, open the DNS Zone Overview page. You will see **4 authoritative Name Servers**:
```
ns1-01.azure-dns.com
ns2-01.azure-dns.net
ns3-01.azure-dns.org
ns4-01.azure-dns.info
```

### Step 3: Delegate the Domain at Your Registrar
1. Log into your domain registrar where you purchased the domain (e.g., GoDaddy, Namecheap, Google Domains).
2. Find the **Custom DNS / Nameservers** section.
3. Replace the default registrar nameservers with Azure's 4 nameservers.
4. Save changes. (DNS propagation takes from 5 minutes up to 24 hours).

---

### Step 4: Add an A Record Pointing to Your Azure VM / Load Balancer
1. Inside your Azure DNS Zone, click **+ Record set**.
2. **Name:** `www` (or leave blank `@` for apex domain).
3. **Type:** `A - IPv4 address`.
4. **TTL:** `3600` (1 hour).
5. **IP address:** Enter the Public IP of your Azure VM or Load Balancer (e.g., `20.120.45.67`).
6. Click **OK**.

### Step 5: Test DNS Resolution
Open your local terminal and test using `nslookup` or `dig`:
```bash
nslookup www.learnazurewithprashant.com
```
**Output:**
```
Non-authoritative answer:
Name:    www.learnazurewithprashant.com
Address: 20.120.45.67
```

---

## 7. Summary Checklist
- [x] Public IPs on backend servers attract brute-force and port scanning attacks.
- [x] Azure Bastion provides secure, browser-based RDP/SSH access without requiring public IPs on VMs.
- [x] Bastion requires a dedicated subnet named exactly `AzureBastionSubnet` with `/26` or larger prefix.
- [x] Azure Public DNS hosts domain records on Microsoft's global high-availability anycast network.
- [x] Private DNS zones provide name resolution within VNets without exposing names or IPs to the internet.
