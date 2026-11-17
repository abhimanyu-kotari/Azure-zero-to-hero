# 10 - Azure Front Door

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:38:36` to `04:43:40`

---

## 1. What is Azure Front Door?

Imagine your web application is deployed in a single Azure datacenter in the United States (`East US`). 
- When a user in New York visits your site, it loads in **30 milliseconds**.
- But when a user in Mumbai, India visits your site, the request must travel halfway around the planet through underwater fiber cables, taking **350+ milliseconds**!
- Even worse, what happens if the entire `East US` region suffers a catastrophic power outage? Your entire business goes offline!

**Azure Front Door** is Microsoft's modern, scalable **Global Content Delivery Network (CDN) and Layer-7 Application Gateway**.

It acts as the single "Global Front Door" for your applications, sitting at the edge of Microsoft's global private network across hundreds of locations worldwide.

---

## 2. How Azure Front Door Works

```
                               [ Global Users ]
                     (US, Europe, India, Australia, Brazil)
                                       |
                                       v
                     +-----------------------------------+
                     |         AZURE FRONT DOOR          |
                     |  (Microsoft Global Edge Network)  |
                     |  - Global Anycast Routing         |
                     |  - Web Application Firewall (WAF) |
                     |  - Edge Static Caching (CDN)      |
                     +-----------------+-----------------+
                                       |
         +-----------------------------+-----------------------------+
         | High-Speed Microsoft Global Fiber Backbone                |
         v                                                           v
+-------------------------------+                           +-------------------------------+
|       PRIMARY REGION          |                           |       SECONDARY REGION        |
|          (East US)            |                           |        (West Europe)          |
|                               |                           |                               |
|   [ Azure Container App / VM] |                           |   [ Azure Container App / VM] |
+-------------------------------+                           +-------------------------------+
```

When a user in Germany visits your website:
1. They connect to the nearest **Microsoft Edge PoP (Point of Presence)** in Frankfurt.
2. Front Door inspects the request and checks the health of all your backend datacenters.
3. Front Door routes the request across **Microsoft's private high-speed fiber-optic network** directly to the fastest healthy region.
4. If the East US region crashes, Front Door **automatically redirects traffic to West Europe in seconds** with zero DNS caching delay!

---

## 3. Core Capabilities of Azure Front Door

### 1. Global Anycast Routing
Users around the world connect to the same single Anycast IP address. Microsoft's network automatically routes each packet to the physically closest edge server, drastically reducing network latency.

### 2. Built-In Content Delivery Network (CDN)
Front Door caches static files (images, logos, CSS, JavaScript, videos) at edge servers worldwide. Subsequent requests are served directly from the edge cache without touching your backend servers.

### 3. Web Application Firewall (WAF) & DDoS Protection
Front Door inspects incoming HTTP traffic for malicious attacks:
- Blocks **SQL Injections** and **Cross-Site Scripting (XSS)**.
- Protects against Layer-7 HTTP flood Distributed Denial of Service (DDoS) attacks before they reach your application.

### 4. SSL/TLS Offloading
Front Door terminates the SSL connection at the edge. Your backend servers don't have to waste CPU processing encryption handshakes.

---

## 4. Cheat Sheet: Load Balancer vs. Application Gateway vs. Front Door

Azure provides three distinct load-balancing services. Here is how to know which one to pick:

| Feature | Azure Load Balancer | Azure Application Gateway | Azure Front Door |
| :--- | :--- | :--- | :--- |
| **OSI Layer** | **Layer 4** (TCP / UDP) | **Layer 7** (HTTP / HTTPS) | **Layer 7** (HTTP / HTTPS) |
| **Scope** | **Regional** (inside 1 region). | **Regional** (inside 1 region). | **Global** (across multiple regions & world). |
| **Routing Decisions** | IP address and port. | URL path (e.g. `/images`, `/api`), Host headers. | Global latency, URL path, geographic location. |
| **Edge Caching / CDN**| No. | No. | **Yes (Full CDN caching)**. |
| **WAF Protection** | No. | Yes (Regional WAF). | **Yes (Global Edge WAF)**. |
| **Best Used For** | Basic TCP traffic between VMs inside a single VNet. | Routing HTTP traffic between microservices in one region. | **Global web apps, multi-region disaster recovery, and CDN acceleration.** |

---

## 5. Summary Checklist
- [x] Front Door is a global Layer 7 load balancer and CDN using Microsoft's Anycast edge network.
- [x] Provides automatic instant multi-region failover for disaster recovery.
- [x] Built-in Web Application Firewall (WAF) stops cyber attacks at the network edge.
- [x] Use Load Balancer for regional Layer 4, App Gateway for regional Layer 7, and Front Door for global applications.
