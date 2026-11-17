# 10 - Global Routing & CDN: Azure Front Door

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:38:36` to `04:43:40`

---

## 1. What is Azure Front Door?

**Azure Front Door** is Microsoft’s modern, global cloud Content Delivery Network (CDN) and enterprise Layer 7 load balancer.

If your users are scattered across the globe (e.g., India, USA, Europe, Australia) and your application servers are located in `East US`:
- Users in India experience high latency (250ms+) because web packets must travel thousands of miles over dozens of unpredictable public ISP hops.
- If the `East US` datacenter experiences an outage, your global business goes dark.

**Azure Front Door solves this by sitting at the edge of Microsoft's global fiber network:**

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
|            | (Healthy Region)                                | (Failover)|
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

1. **Anycast Routing:** Incoming requests hit the closest Microsoft edge Point of Presence (PoP) in milliseconds, entering Microsoft's private global fiber network immediately.
2. **Layer 7 URL Path-Based Routing:**
   - Routes `https://example.com/api/*` -> Backend AKS cluster.
   - Routes `https://example.com/images/*` -> Azure Blob Storage container.
   - Routes `https://example.com/*` -> Azure Static Web App.
3. **Global Multi-Region Failover:** Continuously monitors your primary datacenter. If your US datacenter goes offline, Front Door automatically routes traffic to your European backup with zero manual intervention.
4. **SSL / TLS Offloading:** Performs cryptographic SSL handshakes right at the edge location, saving your backend servers massive CPU cycles.
5. **Integrated Web Application Firewall (WAF):** Inspects incoming packets for OWASP Top 10 vulnerabilities (SQL Injection, Cross-Site Scripting, Malicious Bots) and drops them before they ever reach your servers.

---

## 3. Azure Load Balancer vs. Application Gateway vs. Front Door

A common Azure certification and interview question is choosing the correct load balancing service:

| Feature | Azure Load Balancer | Azure Application Gateway | Azure Front Door |
| :--- | :--- | :--- | :--- |
| **OSI Layer** | **Layer 4** (TCP / UDP) | **Layer 7** (HTTP / HTTPS) | **Layer 7** (HTTP / HTTPS) |
| **Scope** | Regional (Single Azure region) | Regional (Single Azure region) | **Global** (Cross-region / Worldwide) |
| **Protocols** | TCP, UDP | HTTP, HTTPS, WebSockets | HTTP, HTTPS, HTTP/2 |
| **Edge CDN Caching**| No | No | **Yes (Full Global CDN)** |
| **WAF Protection** | No | Yes (Regional) | **Yes (Global Edge WAF)** |
| **Multi-Region Failover**| No | No | **Yes (Instant Global Redirection)**|

---

## 4. Summary Checklist
- [x] Azure Front Door provides global Layer 7 routing and content delivery (CDN).
- [x] Sits at hundreds of edge locations to terminate SSL and cache static assets close to users.
- [x] Provides automated disaster recovery with cross-region backend failover.
- [x] Protects web apps at the perimeter with Web Application Firewall (WAF) rules.
