# Month 8: Azure Cloud Deployment & Architecture Masterclass

Welcome to the complete **Month 8 Azure Cloud Deployment** study guide. These comprehensive, student-friendly notes cover the entire 8-hour masterclass from absolute zero to advanced cloud architecture:

🎬 **Source Tutorial:** [Microsoft AZURE in ONE VIDEO 🔥 For Beginners 2026 [HINDI] | MPrashant](https://youtu.be/M1erI5a2wxQ)  
🎯 **Target Audience:** Beginners, students, freshers, developers, and aspiring Cloud/DevOps engineers.

---

## 📑 Complete 19-Module Study Guide & Video Timelines

| Module | Topic File | Video Timeframe | Core Concepts Covered |
| :---: | :--- | :---: | :--- |
| **01** | [01_cloud_computing_and_azure_basics.md](./01_cloud_computing_and_azure_basics.md) | `00:04:28` - `00:51:43` | Virtualization, Hypervisors (Type 1 & 2), Cloud definitions, IaaS vs PaaS vs SaaS, Azure Global Infrastructure (Regions, AZs, Paired Regions), and Free Account Setup. |
| **02** | [02_compute_azure_virtual_machines.md](./02_compute_azure_virtual_machines.md) | `00:51:43` - `01:44:39` | Azure Compute, VM Sizing Families (B/D/E/F/N series), SSH keys vs passwords, NSG firewall rules, SSH connection via terminal, and deploying an Nginx web server. |
| **03** | [03_azure_vmss_and_load_balancer.md](./03_azure_vmss_and_load_balancer.md) | `01:44:39` - `02:29:44` | High Availability, Vertical vs Horizontal scaling, Azure Virtual Machine Scale Sets (VMSS), Autoscaling rules, Azure Load Balancer (Layer 4), and Health Probes. |
| **04** | [04_azure_virtual_network_vnet.md](./04_azure_virtual_network_vnet.md) | `02:29:44` - `03:10:24` | Azure Virtual Network (VNet), CIDR notation (`/16` vs `/24`), 5 Azure reserved IPs, multi-tier Subnets (Web, App, DB), Network Security Groups (NSGs), and VNet Peering. |
| **05** | [05_azure_bastion_and_dns_zone.md](./05_azure_bastion_and_dns_zone.md) | `03:10:24` - `03:36:13` | Azure Bastion (browser-based secure SSH/RDP without public IPs), Azure Public & Private DNS Zones, and DNS record types (A, AAAA, CNAME, MX, TXT, Alias). |
| **06** | [06_azure_static_web_apps.md](./06_azure_static_web_apps.md) | `03:36:13` - `03:47:55` | Modern static web architecture, automated GitHub Actions CI/CD deployment, Global CDN edge distribution, free SSL/TLS certificates, and PR preview environments. |
| **07** | [07_docker_and_aks_kubernetes.md](./07_docker_and_aks_kubernetes.md) | `03:47:55` - `04:23:32` | Containerization basics, Dockerfile, Container images, Kubernetes orchestration, AKS architecture (Free Control Plane), Pods, Deployments, and LoadBalancer Services. |
| **08** | [08_azure_container_instances_aci.md](./08_azure_container_instances_aci.md) | `04:23:32` - `04:29:29` | Serverless container execution, on-demand execution with zero VM management, per-second billing, public IP & custom DNS labels, restart policies, and batch jobs. |
| **09** | [09_azure_container_apps_aca.md](./09_azure_container_apps_aca.md) | `04:29:29` - `04:38:36` | Serverless microservices, Scale-to-Zero ($0 compute when idle), KEDA autoscaling, built-in HTTPS Ingress, revisions, traffic splitting, and ACI vs ACA vs AKS cheat sheet. |
| **10** | [10_azure_front_door.md](./10_azure_front_door.md) | `04:38:36` - `04:43:40` | Global Layer-7 load balancer & CDN, Microsoft Anycast edge routing, multi-region instant disaster recovery failover, Web Application Firewall (WAF), and SSL offloading. |
| **11** | [11_azure_storage_blob_storage.md](./11_azure_storage_blob_storage.md) | `04:43:40` - `05:25:07` | Storage Accounts, Blob hierarchy (Account -> Container -> Blob), Block vs Append vs Page blobs, Access Tiers (Hot/Cool/Cold/Archive), Redundancy (LRS/ZRS/GRS), and SAS tokens. |
| **12** | [12_identity_and_access_management_iam.md](./12_identity_and_access_management_iam.md) | `05:25:07` - `05:57:26` | Authentication (AuthN) vs Authorization (AuthZ), Microsoft Entra ID (Azure AD), Role-Based Access Control (RBAC), Owner/Contributor/Reader roles, and Scope Inheritance. |
| **13** | [13_azure_table_and_file_storage.md](./13_azure_table_and_file_storage.md) | `05:57:26` - `06:22:08` | Azure Table Storage (schemaless NoSQL, PartitionKey, RowKey), Azure Files (managed SMB/NFS file shares), and persistent volume mounts for VMs and containers. |
| **14** | [14_azure_sql_database.md](./14_azure_sql_database.md) | `06:22:08` - `06:50:47` | Relational databases on Azure (VM vs Managed Instance vs Azure SQL DB), DTU vs vCore models, Serverless auto-pause, firewall security, and Portal Query Editor. |
| **15** | [15_azure_data_factory_adf.md](./15_azure_data_factory_adf.md) | `06:50:47` - `07:19:01` | Data Engineering & ETL pipelines, 6 core ADF components (Linked Services, Datasets, Activities, Pipelines, Integration Runtimes, Triggers), and CSV-to-SQL pipeline. |
| **16** | [16_azure_tags_billing_and_cost_management.md](./16_azure_tags_billing_and_cost_management.md) | `07:19:01` - `07:27:57` | Azure Tags for cost allocation and automation, Cost Analysis forecasting, Azure Budgets with percentage email alerts, and Azure Advisor cost optimizations. |
| **17** | [17_infrastructure_as_code_terraform.md](./17_infrastructure_as_code_terraform.md) | `07:27:57` - `07:45:25` | Infrastructure as Code (IaC), HashiCorp Terraform with Azure (`azurerm`), `init`, `plan`, `apply`, `destroy` lifecycle, declarative HCL syntax, and state management. |
| **18** | [18_infrastructure_as_code_bicep.md](./18_infrastructure_as_code_bicep.md) | `07:45:25` - `07:54:36` | Azure-native IaC, why Bicep replaced ARM JSON templates, transparent transpilation, Day-0 Azure API support, zero state file management, and Bicep vs Terraform. |
| **19** | [19_understanding_ssl_tls_security.md](./19_understanding_ssl_tls_security.md) | `07:54:36` - `08:20:05` | HTTP vs HTTPS, SSL vs TLS, Symmetric vs Asymmetric encryption, step-by-step TLS handshake walkthrough, Certificate Authorities (CAs), and Azure Key Vault SSL. |

---

## 💡 Learning Principles

1. **100% Comprehensive Coverage:** Every single topic from M Prashant's 8-hour masterclass is documented without gaps.
2. **Student-Centric Plain English:** Technical jargon is broken down using real-world analogies (e.g. apartment buildings, postal mailboxes, electricity meters, symphony conductors).
3. **Step-by-Step Hands-On Labs:** Every module contains practical Azure Portal and CLI walkthroughs with commands, configuration parameters, and testing steps.
