# Comprehensive Azure & DevOps Interview Preparation

This comprehensive interview guide synthesizes core architectural, practical, and troubleshooting concepts covered across the entire Azure Zero to Hero course.

---

## Part 1: Core Azure & Cloud Architecture

### Q1: What is the difference between Fault Tolerance, High Availability, and Disaster Recovery?
- **Fault Tolerance:** The ability of a system to continue operating without interruption even when a hardware or software component fails (e.g., redundant power supplies, RAID storage, multi-AZ deployment).
- **High Availability (HA):** Ensuring a system remains operational and accessible with minimal downtime, usually measured in "nines" (e.g., 99.99% uptime via load balancers and scale sets).
- **Disaster Recovery (DR):** The strategy and process of restoring critical infrastructure and data after a catastrophic event (e.g., region-wide outage) measured by RTO (Recovery Time Objective) and RPO (Recovery Point Objective).

### Q2: What is an Availability Zone vs an Availability Set?
- **Availability Set:** Protects VMs against hardware failures within a single datacenter by distributing VMs across different **Fault Domains** (racks with separate power and networking) and **Update Domains** (groups of VMs rebooted during scheduled host maintenance).
- **Availability Zone (AZ):** Physically separate datacenters within an Azure region, each with independent power, cooling, and networking. Provides resilience against datacenter-level disasters.

---

## Part 2: Networking & Security

### Q3: How do you securely connect two Virtual Networks (VNets) in Azure?
- **VNet Peering:** Connects two VNets using Microsoft's private backbone network with high bandwidth and ultra-low latency. Traffic does not traverse the public internet.
- **VPN Gateway:** Uses encrypted IPsec tunnels over the public internet. Best for connecting on-premises networks to Azure or connecting VNets across different cloud providers.

### Q4: Why would you use Azure Bastion instead of a standard Jumpbox VM with a Public IP?
- A standard Jumpbox requires opening port 22 (SSH) or 3389 (RDP) to the public internet, leaving it vulnerable to brute-force attacks and port scanning.
- **Azure Bastion** is a fully managed PaaS service provisioned directly inside your VNet. It provides secure, TLS-encrypted RDP and SSH connectivity directly within the Azure Portal over port 443, eliminating the need for public IPs on any virtual machines.

---

## Part 3: Compute & Kubernetes (AKS)

### Q5: How do you handle secrets securely in Azure Kubernetes Service (AKS)?
- Avoid storing raw secrets in Git or plaintext ConfigMaps.
- **Best Practice:** Use **Azure Key Vault Provider for Secrets Store CSI Driver**. Secrets stored securely in Azure Key Vault are mounted into pods as local volume files using Azure Workload Identity (federated credentials). Secrets never leave memory and can be automatically synced into native Kubernetes secrets.

### Q6: What is the purpose of readiness and liveness probes in Kubernetes?
- **Liveness Probe:** Detects if an application is deadlocked or in a broken state. If the probe fails, kubelet kills the container and restarts it according to its restart policy.
- **Readiness Probe:** Detects if an application is ready to accept user network traffic (e.g., after loading configuration or initializing database connections). If it fails, the pod is temporarily removed from the Kubernetes Service load balancer backend pool.

---

## Part 4: CI/CD & DevOps Automation

### Q7: What are the differences between Azure Pipelines and GitHub Actions?
- **Azure Pipelines:** Enterprise-grade CI/CD engine supporting both YAML and visual Classic editors, advanced deployment environments, release gates, and tight integration with Azure Boards and Azure Artifacts.
- **GitHub Actions:** Developer-first CI/CD built natively into GitHub with an enormous public marketplace of reusable open-source actions.

### Q8: How do you prevent broken builds from reaching the production branch in Azure DevOps?
- Implement **Branch Policies** on the `main` branch in Azure Repos:
  1. Require a minimum number of peer reviewers on Pull Requests.
  2. Enforce **Build Validation**—an automated CI pipeline must successfully compile code and pass unit tests before merging is permitted.
  3. Require all comment threads to be marked as resolved.

---

## Part 5: Infrastructure as Code (Terraform)

### Q9: Why is Remote State management in Azure Blob Storage critical when using Terraform in a team?
- By default, Terraform stores state locally in `terraform.tfstate`. If multiple engineers run Terraform locally, state files get out of sync, causing resource corruption or accidental deletions.
- Storing state remotely in **Azure Blob Storage** ensures a single source of truth.
- Azure Blob Storage provides **native state locking** (via blob leases), preventing race conditions when two team members or CI pipelines attempt to apply changes simultaneously.

---

## Part 6: Scenario-Based Troubleshooting

### Scenario: An application pod on AKS cannot connect to a private Azure SQL database or backend VM. How do you troubleshoot?
1. **DNS Resolution:** Exec into the pod (`kubectl exec -it <pod> -- sh`) and test DNS resolution using `nslookup <hostname>` or `curl`.
2. **Network Security Groups (NSGs):** Check NSG rules associated with the AKS subnet and the destination subnet to verify inbound port 1433/database port is not blocked.
3. **Azure Network Watcher:** Run **IP Flow Verify** from the AKS node to the target database IP/port to see which specific NSG rule allowed or denied the packet.
4. **Firewall / Route Tables:** Check if a User Defined Route (UDR `0.0.0.0/0`) directs traffic through an Azure Firewall that is dropping the packets.
5. **Private Endpoints:** If using Azure Private Endpoint, confirm Private DNS Zone is properly linked to the AKS Virtual Network.
