# Introduction to Azure Serverless Computing

## What is Serverless Computing?

Serverless computing is a cloud-native development model that allows developers to build and run applications without managing, provisioning, or maintaining infrastructure. 

Despite the name "serverless," physical and virtual servers still exist; however, the cloud provider (Microsoft Azure) completely abstracts server management away from the developer.

---

## Architectural Comparison: IaaS vs PaaS vs Containers vs Serverless

| Dimension | IaaS (Virtual Machines) | PaaS (App Service) | Containers (AKS / ACI) | Serverless (Azure Functions) |
| :--- | :--- | :--- | :--- | :--- |
| **Server Management** | Customer manages OS, patches, networking | Managed by Azure | Managed worker nodes / cluster | **Zero server management** |
| **Scaling** | Manual or VMSS autoscale (minutes) | Auto-scale rules (minutes) | HPA + Cluster Autoscaler (1-2 mins) | **Instant, sub-second elastic scaling** |
| **Idle Billing** | Pay 24/7 for VM, even if idle | Pay 24/7 for App Service Plan tier | Pay for running cluster nodes | **Pay $0 when idle (scale to zero)** |
| **Execution Model** | Long-running background processes | Continuous web application | Continuous microservices | **Event-driven, short-lived executions** |
| **Pricing Unit** | Per second/hour VM size | Per hour App Service tier | Per vCPU/RAM per node | **Per execution count & GB-seconds** |

---

## The Azure Serverless Ecosystem

Microsoft Azure offers a suite of complementary serverless services:

```
+----------------------------------------------------------------------------------+
|                            AZURE SERVERLESS PLATFORM                             |
|                                                                                  |
|   +--------------------------+    +--------------------------+                   |
|   |     Azure Functions      |    |     Azure Logic Apps     |                   |
|   |  (Event-Driven Compute)  |    |  (No-Code Workflows)     |                   |
|   |  Python, C#, Node, Go    |    |  400+ Connectors         |                   |
|   +-------------+------------+    +-------------+------------+                   |
|                 ^                               ^                                |
|                 |                               |                                |
|                 +---------------+---------------+                                |
|                                 |                                                |
|                   +-------------+-------------+                                  |
|                   |     Azure Event Grid      |                                  |
|                   |  (Reactive Pub/Sub Router)|                                  |
|                   +-------------+-------------+                                  |
|                                 ^                                                |
|             +-------------------+-------------------+                            |
|             |                   |                   |                            |
|       [ Azure Blob ]     [ Resource Group ]   [ Azure Alerts ]                   |
+----------------------------------------------------------------------------------+
```

1. **Azure Functions (Compute):** Run small pieces of code (functions) in response to events without managing an application server.
2. **Azure Logic Apps (Orchestration):** Visual workflow designer with over 400 pre-built enterprise connectors (Office 365, Salesforce, SAP, GitHub, ServiceNow).
3. **Azure Event Grid (Routing):** Highly scalable, fully managed event broker that uses publish-subscribe logic to route events from Azure resources to handlers with sub-second delivery.
4. **Azure Event Hubs (Data Streaming):** Serverless big data streaming engine capable of ingesting millions of events per second (e.g., IoT telemetry).
