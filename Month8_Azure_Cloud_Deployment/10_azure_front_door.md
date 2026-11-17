# 10 - Global Routing & CDN: Azure Front Door

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:38:36` to `04:43:40`

---

## 1. The Global Latency & Availability Problem

If your users are distributed across the globe (e.g., India, USA, Europe, Australia) and your application servers are hosted exclusively in a single region like `East US`:
- Users in India experience severe latency (250ms+) because web packets must traverse thousands of miles over multiple unpredictable public ISP hops.
- If the `East US` datacenter experiences an unexpected power outage or fiber cut, your global business goes dark.

**Azure Front Door solves this by positioning your entry point at the edge of Microsoft's global fiber network:**

```
+--------------------------------------------------------------------------+
|                       AZURE FRONT DOOR ARCHITECTURE                      |
+--------------------------------------------------------------------------+
|                                                                          |
|     Global Users (London, Tokyo, Mumbai, New York)                       |
|                             |                                            |
|                             v Anycast DNS                                |
|   +------------------------------------------------------------------+   |
|   |   MICROSOFT GLOBAL EDGE LOCATIONS (Hundreds of Edge PoPs)        |   |
|   |   - SSL/TLS Offloading at the Edge                               |   |
|   |   - Web Application Firewall (WAF) blocks SQLi & XSS attacks     |   |
|   |   - Static Asset Caching (CDN)                                   |   |
|   +---------------------------------+--------------------------------+   |
|                                     |                                    |
|              +======================+======================+             |
|              | Microsoft Private Global WAN Fiber Backbone |             |
|              +======================+======================+             |
|                                     |                                    |
|            +------------------------+------------------------+           |
|            | (Healthy Primary)                               | (Failover)|
|            v                                                 v           |
|   +--------------------------+                     +-------------------+ |
|   | Primary Backend: East US |                     | Secondary: Europe | |
|   | (App Service / AKS / VM) |                     | (Disaster Recovery| |
|   +--------------------------+                     +-------------------+ |
|                                                                          |
+--------------------------------------------------------------------------+
```

---

## 2. Core Capabilities of Azure Front Door

1. **Anycast Routing:** Incoming requests hit the nearest Microsoft edge Point of Presence (PoP) in milliseconds, entering Microsoft's private global fiber network immediately.
2. **Layer 7 URL Path-Based Routing:**
   - Routes `https://example.com/api/*` -> Backend AKS cluster.
   - Routes `https://example.com/images/*` -> Azure Blob Storage container.
   - Routes `https://example.com/*` -> Azure Static Web App.
3. **Global Multi-Region Failover:** Continuously monitors your primary datacenter. If your US datacenter goes offline, Front Door automatically routes traffic to your European backup with zero manual intervention.
4. **SSL / TLS Offloading:** Performs cryptographic SSL handshakes right at the edge location, saving backend servers massive CPU cycles.
5. **Integrated Web Application Firewall (WAF):** Inspects incoming packets for OWASP Top 10 vulnerabilities (SQL Injection, Cross-Site Scripting, Malicious Bots) and drops them before they ever reach your servers.
6. **Dynamic Site Acceleration (DSA):** Splits TCP handshakes at the edge, eliminating round-trip latency across the public internet.

---

## 3. Azure Load Balancer vs. Application Gateway vs. Front Door

A standard Azure certification and interview question is selecting the correct load balancing service:

| Feature | Azure Load Balancer | Azure Application Gateway | Azure Front Door |
| :--- | :--- | :--- | :--- |
| **OSI Layer** | **Layer 4** (TCP / UDP) | **Layer 7** (HTTP / HTTPS) | **Layer 7** (HTTP / HTTPS) |
| **Scope** | Regional (Single Azure region) | Regional (Single Azure region) | **Global** (Cross-region / Worldwide) |
| **Protocols** | TCP, UDP | HTTP, HTTPS, WebSockets | HTTP, HTTPS, HTTP/2 |
| **Edge CDN Caching**| No | No | **Yes (Full Global CDN)** |
| **WAF Protection** | No | Yes (Regional) | **Yes (Global Edge WAF)** |
| **Multi-Region Failover**| No | No | **Yes (Instant Global Redirection)**|

---

## 4. Hands-on Lab: Configuring Azure Front Door

### Step 1: Create Azure Front Door Profile
1. Search for **Front Door and CDN profiles** in the Azure Portal and click **+ Create**.
2. **Compare offerings:**
   - **Azure Front Door Standard:** Content delivery, Layer 7 routing, basic security.
   - **Azure Front Door Premium:** Adds managed WAF rule sets, bot protection, and private link origins.
   - Choose **Azure Front Door Standard** -> Click **Continue to create Front Door**.

---

### Step 2: Configure Endpoints, Origins & Routing
1. **Basics Tab:**
   - **Subscription:** Select your subscription.
   - **Resource group:** `rg-routing-demo`.
   - **Profile name:** `fd-global-service`.
2. **Configuration Settings:**
   - **Endpoint name:** `myglobalapp-endpoint` (Generates `myglobalapp-endpoint.azurefd.net`).
   - **Origin type:** Select **App Services**, **Storage**, or **Custom** (Enter Public IP/FQDN of your backend servers).
   - **Origin host name:** Point to your primary backend VMSS or App Service.
   - **Caching:** Enable caching if serving static assets.
3. Click **Review + create** -> **Create**.
4. Front Door propagates across Microsoft's global edge network in 3–5 minutes.

---

## 5. Troubleshooting Front Door Issues

| Problem | Cause | Resolution |
| :--- | :--- | :--- |
| Front Door returns `400 Bad Request (Invalid Host Name)`. | Backend server expects the original host header. | In Front Door route settings, set **Origin host header** to match the backend server's expected domain. |
| Health probes fail on origin group. | Origin server firewall blocks Microsoft edge IPs. | Ensure backend NSG permits traffic from the `AzureFrontDoor.Backend` service tag. |
| Changes to static assets not reflected. | Edge CDN caching is active. | Go to your Front Door endpoint in the portal and click **Purge cache** to clear stale assets globally. |

---

## 6. Summary Checklist
- [x] Azure Front Door delivers global Layer 7 routing and content delivery (CDN).
- [x] Sits at hundreds of edge locations to terminate SSL and cache static assets close to users.
- [x] Provides automated disaster recovery with cross-region backend failover.
- [x] Protects web apps at the perimeter with Web Application Firewall (WAF) rules.
