# 05 - Azure Bastion & Azure DNS Zones

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `03:10:24` to `03:36:13`

---

## Part 1: Azure Bastion Host

### 1. The Security Problem: How Do You Connect to Private VMs?
In enterprise cloud architecture, production servers and database virtual machines **must never have public IP addresses**. If a database VM has a public IP and open SSH port (22), automated hacker bots around the globe will scan it 24/7 trying brute-force password attacks.

**The Dilemma:** If your VMs have only private IP addresses (like `10.0.2.4`), how do your developers and administrators securely log in to fix bugs or run updates?

```
THE OLD RISKY WAY: JUMPBOX WITH PUBLIC IP
[ Admin Laptop ] ---> (Public Internet) ---> [ Jumpbox VM with Public IP ] ---> [ Private DB VM ]
                                                    ^
                                                    |
                                      DANGER: Exposed to port scanning & attacks!
```

---

### 2. What is Azure Bastion?
**Azure Bastion** is a fully managed PaaS (Platform as a Service) resource that you deploy directly inside your Virtual Network. 

It lets you connect securely to any Linux VM (via SSH) or Windows VM (via RDP) **directly inside your web browser over secure HTTPS (Port 443)**!

```
THE SECURE AZURE BASTION WAY:
[ Admin Laptop ] 
       |
       | Encrypted HTTPS (Port 443) via Web Browser
       v
[ Azure Portal ] 
       |
       v
[ Azure Bastion Host ] (Inside private "AzureBastionSubnet")
       |
       | Internal Private Network SSH / RDP
       v
[ Private Backend VMs ] (Zero Public IPs needed!)
```

### Why DevOps Engineers Love Azure Bastion:
1. **Zero Public IPs Required:** Your virtual machines remain 100% private.
2. **No Client Software Needed:** You don't need PuTTY, SSH clients, or RDP software installed on your laptop. You just open Chrome or Edge, navigate to the Azure Portal, click *Connect via Bastion*, and a full interactive terminal/desktop opens inside the browser tab!
3. **Protected Against Port Scanners:** Because ports 22 and 3389 are closed to the public internet, port-scanning bots cannot detect your servers.
4. **Centralized Auditing:** Integration with Azure Monitor tracks exactly who logged in and when.

---

## Part 2: Azure DNS Zones

### 1. What is DNS (Domain Name System)?
Computers communicate using numerical IP addresses (like `20.120.45.67`), but humans remember names (like `google.com` or `mycompany.com`).

The **Domain Name System (DNS)** is the "phonebook of the internet." When you type a web address into your browser, DNS translates that friendly name into the exact computer IP address hosting the website.

---

### 2. What is an Azure DNS Zone?
An **Azure DNS Zone** hosts the DNS records for your domain name using Microsoft's ultra-fast, globally distributed Anycast network of DNS name servers.

> **Note:** Azure DNS is a **hosting** service for DNS records. You can purchase your domain name anywhere (like GoDaddy, Namecheap, or Azure App Service Domains) and point its Name Servers (NS) to Azure DNS to manage your records.

---

### 3. Public DNS Zones vs. Private DNS Zones

| Feature | Public DNS Zone | Private DNS Zone |
| :--- | :--- | :--- |
| **Audience** | Anyone in the world on the public internet. | Only Virtual Machines connected to your private Azure VNet. |
| **Resolution** | Resolves external domains (e.g. `www.mycloudapp.com` -> Public IP). | Resolves internal hostnames (e.g. `db.internal.corp` -> `10.0.2.4`). |
| **Internet Visibility**| Publicly queryable via `nslookup` or `dig`. | Invisible and inaccessible to the outside world. |
| **Use Case** | Routing customer traffic to your website or load balancer. | Letting backend microservices find databases without hardcoding IPs. |

---

### 4. Essential DNS Record Types Every Student Must Know

| Record Type | Name | Purpose | Example |
| :---: | :--- | :--- | :--- |
| **A** | Address Record | Maps a domain name directly to an **IPv4 address**. | `api.mycompany.com` &rarr; `20.120.45.67` |
| **AAAA** | IPv6 Address | Maps a domain name to a 128-bit **IPv6 address**. | `ipv6.mycompany.com` &rarr; `2001:db8::1` |
| **CNAME** | Canonical Name | Maps an alias name to another domain name (redirect). | `blog.mycompany.com` &rarr; `myblog.azurewebsites.net` |
| **MX** | Mail Exchange | Directs emails sent to your domain to mail servers. | `@` &rarr; `mail.protection.outlook.com` |
| **TXT** | Text Record | Holds arbitrary text for domain verification (SPF, Google/Office365 ownership). | `MS=ms12345678` |
| **Alias** | Azure Alias | Dynamically links your domain directly to an Azure Public IP or Traffic Manager. | Automatically updates if the Azure resource IP changes! |

---

## 5. Hands-on Walkthrough: Setting Up Azure DNS

In the video, M Prashant demonstrates configuring a DNS record:

### Step 1: Create a Public DNS Zone
1. In the Azure Portal, search for **DNS zones** -> Click **Create**.
2. Resource Group: `rg-networking-demo`.
3. Name: Enter your purchased domain name (e.g., `studentcloudlab.com`).
4. Click **Review + Create** -> **Create**.

### Step 2: Update Name Servers at Your Domain Registrar
1. Open your new DNS Zone in the portal.
2. Locate the 4 **Name servers** provided by Azure:
   - `ns1-01.azure-dns.com`
   - `ns2-01.azure-dns.net`
   - `ns3-01.azure-dns.org`
   - `ns4-01.azure-dns.info`
3. Log into your domain registrar (GoDaddy, Namecheap) and replace their default nameservers with these 4 Azure addresses.

### Step 3: Add an "A Record"
1. In the Azure DNS Zone dashboard, click **+ Record set**.
2. **Name:** `www`
3. **Type:** `A - IPv4 address`
4. **TTL (Time to Live):** `3600` (1 hour)
5. **IP address:** Paste the Public IP of your Azure Load Balancer or VM (e.g. `20.120.45.67`).
6. Click **OK**.

Now, typing `http://www.studentcloudlab.com` in any browser around the globe resolves to your Azure cloud server!

---

## 6. Summary Checklist
- [x] Azure Bastion provides secure, browser-based SSH and RDP over HTTPS without public IPs on VMs.
- [x] Dedicated subnet named `AzureBastionSubnet` is required for Bastion deployment.
- [x] Azure DNS hosts domain records on Microsoft's fast Anycast global network.
- [x] Use Public DNS for internet-facing sites; use Private DNS for internal VNet server resolution.
- [x] Understand core records: A (name to IP), CNAME (name to name), MX (email routing), TXT (verification).
