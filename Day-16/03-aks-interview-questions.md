# Azure Kubernetes Service (AKS) Interview Q&A

### 1. What happens behind the scenes when you create an AKS cluster?
**Answer:**
Azure provisions a dedicated, highly available Kubernetes control plane (`kube-apiserver`, `etcd`, `kube-controller-manager`, `kube-scheduler`) inside a Microsoft-managed Azure subscription. In your customer subscription, Azure creates:
- A Virtual Network (or joins an existing one).
- A Managed Resource Group (prefixed with `MC_<resourceGroup>_<clusterName>_<location>`).
- An Azure Virtual Machine Scale Set (VMSS) containing the worker nodes.
- A Network Security Group (NSG), Public IP, and Load Balancer for ingress/egress.

---

### 2. What is the difference between Kubenet and Azure CNI? When would you choose one over the other?
**Answer:**
- **Kubenet:** Pods receive private IP addresses from an internal overlay CIDR block that does not belong to your Azure VNet. Nodes perform NAT for outbound communication. Choose Kubenet when you have limited IP address space in your corporate VNet.
- **Azure CNI:** Every pod receives a real, routable IP address directly from the Azure VNet subnet. This provides higher network performance and allows other Azure resources to talk directly to pods without NAT. Choose Azure CNI for enterprise environments requiring direct private network communication, though you must carefully plan subnet sizing.

---

### 3. What is the difference between Horizontal Pod Autoscaler (HPA) and Cluster Autoscaler (CA)?
**Answer:**
- **HPA (Horizontal Pod Autoscaler):** Operates at the Kubernetes layer. It scales the number of **pod replicas** up or down based on metrics like CPU utilization, memory, or custom Prometheus metrics.
- **Cluster Autoscaler (CA):** Operates at the infrastructure layer. When HPA scales up pods and there is insufficient CPU/memory on existing worker nodes (pods enter `Pending` state), Cluster Autoscaler automatically provisions additional **VM instances** in the VMSS. When nodes are underutilized for a period (e.g., 10 minutes), CA drains and deletes them.

---

### 4. Why should you separate System Node Pools from User Node Pools?
**Answer:**
- **System Node Pool:** Hosts critical Kubernetes operational components (`CoreDNS`, `metrics-server`, `konnectivity-agent`, CSI drivers). If user workloads crash due to memory leaks or consume 100% CPU on these nodes, the entire cluster's DNS and control plane connectivity could fail.
- **User Node Pool:** Runs customer business applications. By isolating them on separate node pools with Kubernetes taints and tolerations, application failures cannot destabilize cluster operations.

---

### 5. How do you secure pod access to Azure resources without hardcoding credentials?
**Answer:**
Using **Azure Workload Identity**:
1. Enable OIDC Issuer and Workload Identity on the AKS cluster.
2. Create a User-Assigned Managed Identity in Azure and assign it required RBAC roles (e.g., Key Vault Secrets Officer or Storage Blob Data Contributor).
3. Create a Kubernetes Service Account and annotate it with the client ID of the Azure Managed Identity.
4. Establish a federated identity credential between the AKS OIDC issuer and the Managed Identity.
5. Pods running with this Service Account automatically exchange short-lived Kubernetes projection tokens for Azure AD access tokens without passwords or stored secrets.

---

### 6. What is a Private AKS Cluster and how does it improve security?
**Answer:**
In a standard AKS cluster, the `kube-apiserver` has a public FQDN and IP address (secured by TLS and RBAC). In a **Private AKS Cluster**, the API server endpoint is not exposed to the public internet; instead, it receives a Private IP address via an Azure Private Endpoint in your VNet. Access to `kubectl` is only possible from within the VNet, through a peered VNet, a Jumpbox (Bastion), or an ExpressRoute/VPN from on-premises.

---

### 7. How does AKS handle node operating system security updates and patching?
**Answer:**
Microsoft automatically applies OS security patches (e.g., Ubuntu/Azure Linux CVE patches) nightly to node images. However, kernel updates requiring a reboot are not rebooted automatically by default. DevOps engineers use **Kured (Kubernetes Reboot Daemon)** or configure **AKS Node OS Auto-Upgrade channels** (e.g., `NodeImage` channel) with Maintenance Windows to safely cordon, drain, reboot, or reimage nodes without application downtime.
