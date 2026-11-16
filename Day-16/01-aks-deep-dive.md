# Azure Kubernetes Service (AKS) Deep Dive

## What is AKS?

Azure Kubernetes Service (AKS) is a fully managed container orchestration service provided by Microsoft Azure. AKS simplifies deploying, managing, and scaling containerized applications using open-source Kubernetes without the operational overhead of managing the Kubernetes control plane.

---

## AKS Architecture: Control Plane vs Data Plane

Kubernetes consists of two primary operational layers:

```
+---------------------------------------------------------------------------------+
|                        AZURE MANAGED CONTROL PLANE (FREE)                       |
|                                                                                 |
|  +-------------------+  +--------------------+  +----------------------------+  |
|  |  kube-apiserver   |  |   kube-scheduler   |  |  kube-controller-manager   |  |
|  +-------------------+  +--------------------+  +----------------------------+  |
|                                                                                 |
|                     +---------------------------------------+                   |
|                     |     etcd (Distributed Key-Value DB)   |                   |
|                     +---------------------------------------+                   |
|                                                                                 |
|  * Fully managed, backed by Microsoft SLA, auto-upgraded, highly available.     |
+----------------------------------------+----------------------------------------+
                                         | Secure Tunnel
                                         v
+---------------------------------------------------------------------------------+
|                        CUSTOMER DATA PLANE (WORKER NODES)                       |
|                                                                                 |
|  +----------------------------------+    +----------------------------------+   |
|  |        System Node Pool          |    |         User Node Pool           |   |
|  |   (VMSS: e.g. Standard_D2s_v5)   |    |   (VMSS: e.g. Standard_D4s_v5)   |   |
|  |                                  |    |                                  |   |
|  |  - CoreDNS                       |    |  - Voting App Pods               |   |
|  |  - Metrics Server                |    |  - Payment Service Pods          |   |
|  |  - CSI Storage Drivers           |    |  - Redis & Database Pods         |   |
|  |  - Kube-Proxy & Kubelet          |    |  - Kube-Proxy & Kubelet          |   |
|  +----------------------------------+    +----------------------------------+   |
|                                                                                 |
|  * Runs on Azure Virtual Machine Scale Sets (VMSS). Customer pays only for VMs. |
+---------------------------------------------------------------------------------+
```

### 1. The Control Plane (Managed by Azure)
- **kube-apiserver:** The front door of Kubernetes; exposes the Kubernetes API and validates requests.
- **etcd:** Distributed, consistent key-value store holding the complete state and configuration of the cluster.
- **kube-scheduler:** Watches for newly created pods with no assigned node and assigns them to suitable worker nodes.
- **kube-controller-manager:** Runs controller loops (Node Controller, Replication Controller, Endpoints Controller).
- **Management:** Azure completely manages, monitors, patches, and backs up the control plane at **zero additional charge** (for the standard tier, or backed by a 99.95% SLA with the Uptime SLA tier).

### 2. The Data Plane (Worker Nodes)
- Worker nodes execute user workloads. They run:
  - **containerd:** The container runtime that runs containers.
  - **kubelet:** The node agent that ensures containers described in PodSpecs are running and healthy.
  - **kube-proxy:** Manages network routing and IP translation (iptables/IPVS) for Kubernetes Services.

---

## Node Pools in AKS

AKS uses **Azure Virtual Machine Scale Sets (VMSS)** to group VMs into node pools:

1. **System Node Pool:**
   - Dedicated to critical system pods (`kube-system` namespace) such as CoreDNS, metrics-server, CSI storage drivers, and Azure CNI agents.
   - Tainted to prevent customer user workloads from displacing critical cluster components.
   - Minimum recommendation: At least 2-3 nodes in production for high availability.

2. **User Node Pool:**
   - Dedicated solely to customer application pods and microservices.
   - Can use specialized hardware: compute-optimized, memory-optimized, or GPU-enabled VMs (e.g., NC-series).
   - Supports scale-to-zero (`min_count = 0`) to optimize costs during off-peak hours.

---

## AKS Networking Models

| Feature | Kubenet (Basic) | Azure CNI (Advanced) | Azure CNI Overlay |
| :--- | :--- | :--- | :--- |
| **IP Allocation** | Pods get private IPs from an internal overlay network (NAT). | Every pod receives a real private IP from your Azure VNet subnet. | Pods get IPs from a private CIDR; nodes get Azure VNet IPs. |
| **VNet IP Consumption** | Very low (only worker nodes consume VNet IPs). | Very high (must pre-allocate subnet IPs for max pods per node). | Low (combines VNet performance with Kubenet IP conservation). |
| **Direct VNet Routing** | No, requires NAT through the worker node. | Yes, pods are directly addressable from other VNets / On-prem. | Pods route through node-level overlay encapsulation. |
| **Network Policies** | Calico | Azure Network Policies or Calico | Azure Network Policies or Calico |

---

## Autoscaling in AKS

1. **Horizontal Pod Autoscaler (HPA):** Scales the number of pod replicas up or down based on CPU, memory, or custom metrics (e.g., HTTP request rate).
2. **Cluster Autoscaler (CA):** Watches for pods that cannot be scheduled due to insufficient compute resources and automatically scales the number of VMs in the VMSS up. When nodes remain idle, it scales down.
3. **KEDA (Kubernetes Event-driven Autoscaling):** Event-driven autoscaling for pods based on external events (e.g., Azure Service Bus queue length, Kafka lag, Redis queue depth).
