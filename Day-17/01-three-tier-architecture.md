# Deploying a Three-Tier E-Commerce Architecture on AKS (Project 4)

## What is Three-Tier Architecture?

A Three-Tier Architecture is a client-server software architecture pattern in which the functional process logic, data access, computer data storage, and user interface are developed and maintained as independent modules on separate platforms:

```
+-----------------------------------------------------------------------------------+
|                        TIER 1: PRESENTATION TIER (FRONTEND)                       |
|                                                                                   |
|  [ End User Browser ] ----> [ Azure Load Balancer / Public IP ]                   |
|                                      |                                            |
|                                      v                                            |
|                           [ NGINX Ingress Controller ]                            |
|                                      |                                            |
|                                      v                                            |
|                        [ Web Frontend Service (Pod) ]                             |
+--------------------------------------+--------------------------------------------+
                                       | HTTP / REST API calls
                                       v
+-----------------------------------------------------------------------------------+
|                        TIER 2: APPLICATION TIER (BUSINESS LOGIC)                  |
|                                                                                   |
|  +--------------------+   +--------------------+   +---------------------------+  |
|  |    User Service    |   |    Cart Service    |   |     Catalogue Service     |  |
|  |     (Node.js)      |   |      (Python)      |   |          (Go)             |  |
|  +---------+----------+   +---------+----------+   +-------------+-------------+  |
|            |                        |                            |                |
|  +---------+----------+   +---------+----------+   +-------------+-------------+  |
|  |  Payment Service   |   |  Shipping Service  |   |      Ratings Service      |  |
|  |     (Python)       |   |       (Java)       |   |          (PHP)            |  |
|  +--------------------+   +--------------------+   +---------------------------+  |
+--------------------------------------+--------------------------------------------+
                                       | Internal DB Connections / AMQP
                                       v
+-----------------------------------------------------------------------------------+
|                        TIER 3: DATA TIER (DATABASES & QUEUES)                     |
|                                                                                   |
|  +--------------------+   +--------------------+   +---------------------------+  |
|  |   MongoDB (Users   |   |   Redis (Session   |   |  RabbitMQ (Order Events   |  |
|  |    & Catalogue)    |   |     & Cart)        |   |       & Dispatch)         |  |
|  +--------------------+   +--------------------+   +---------------------------+  |
+-----------------------------------------------------------------------------------+
```

---

## The 8 Microservices & 2 Databases Breakdown

In our E-Commerce application, the system is decomposed into 8 discrete microservices and 2 databases:

| Component | Tier | Technology | Description |
| :--- | :--- | :--- | :--- |
| **Web** | Presentation | Node.js / HTML5 | Client-facing storefront interface. |
| **User** | Application | Node.js | User authentication, registration, profiles. |
| **Catalogue** | Application | Go | Product catalog search, categories, inventory. |
| **Cart** | Application | Python | Manages shopping cart items and active sessions. |
| **Payment** | Application | Python | Processes mock payment transactions. |
| **Shipping** | Application | Java (Spring) | Calculates shipping costs and coordinates orders. |
| **Ratings** | Application | PHP | Customer reviews, ratings, and feedback. |
| **Dispatch** | Application | Go | Background order queue processor. |
| **MongoDB** | Data | NoSQL Database | Stores product catalog and user profile records. |
| **Redis** | Data | In-Memory Cache | Fast ephemeral storage for cart and user sessions. |

---

## Inter-Service Communication

1. **Service Discovery:** Pods discover each other using **Kubernetes CoreDNS**. For example, the `web` service contacts `catalogue` at `http://catalogue.ecommerce.svc.cluster.local:8080`.
2. **ClusterIP Services:** Backend microservices are never exposed to the public internet. They only have internal `ClusterIP` services.
3. **Ingress Controller:** Exposes only the `web` frontend service and relevant API gateways to external internet traffic on port 80/443.
