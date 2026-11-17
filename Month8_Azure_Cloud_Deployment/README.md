# Month 8: Azure Cloud Deployment & Fundamentals Study Guide

Welcome to the **Month 8 Azure Cloud Deployment** study repository. These comprehensive, student-friendly notes are synthesized directly from the masterclass video by **M Prashant**:

🎬 **Source Tutorial:** [Microsoft AZURE in ONE VIDEO 🔥 For Beginners [HINDI] | MPrashant](https://youtu.be/M1erI5a2wxQ)  
🎯 **Target Audience:** Beginners, students, freshers, and developers preparing for Cloud and DevOps deployment workflows.

---

## 📑 Core Syllabus Index & Video Timelines

This study guide focuses on the 7 core foundational and deployment modules covered in the first ~5 hours of the masterclass:

| Module | Topic File | Video Timeframe | Core Concepts Covered |
| :---: | :--- | :---: | :--- |
| **01** | [01_cloud_computing_and_azure_basics.md](./01_cloud_computing_and_azure_basics.md) | `00:04:28` - `00:51:43` | Virtualization, Hypervisors (Type 1 & 2), Cloud definitions, IaaS vs PaaS vs SaaS, Azure Global Infrastructure (Regions, AZs), and Free Account Setup. |
| **02** | [02_compute_azure_virtual_machines.md](./02_compute_azure_virtual_machines.md) | `00:51:43` - `01:44:39` | Laptop analogy, VM sizing (B/D/E/F series), SSH keys, Apache install script, SCP file transfer from local laptop, Cloud Shell, 6-step port verification lab (`netstat -tulnp`, `ss`, `curl`, `nc`), NSG Port 80 experiment, and data disk mounting. |
| **03** | [03_docker_and_containerization_basics.md](./03_docker_and_containerization_basics.md) | `03:47:55` - `04:23:32` | Container concepts, Shipping container analogy, Containers vs VMs, Docker architecture, Dockerfile recipe, Docker CLI lifecycle, and why Kubernetes (AKS) orchestration is needed. |
| **04** | [04_azure_container_instances_aci.md](./04_azure_container_instances_aci.md) | `04:23:32` - `04:29:29` | Serverless container execution, on-demand execution with zero VM management, per-second billing, public IP & custom DNS name labels, restart policies, and Azure Files volume mounting. |
| **05** | [05_azure_container_apps_aca.md](./05_azure_container_apps_aca.md) | `04:29:29` - `04:38:36` | Modern microservices on Azure, Scale-to-Zero cost savings ($0 when idle), Envoy Ingress with free SSL, KEDA autoscaling, revisions, traffic splitting canary releases (80/20), and ACI vs ACA vs AKS comparison. |
| **06** | [06_azure_blob_storage.md](./06_azure_blob_storage.md) | `04:43:40` - `05:25:07` | Unstructured object storage, Storage Account hierarchy, Block/Append/Page blobs, Access Tiers (Hot/Cool/Cold/Archive rehydration), Redundancy (LRS/ZRS/GRS), time-limited SAS tokens, and AzCopy. |
| **07** | [07_identity_and_access_management_iam.md](./07_identity_and_access_management_iam.md) | `05:25:07` - `05:57:26` | Authentication vs Authorization (Airport & Passport analogy), Microsoft Entra ID (Azure AD), Resource Hierarchy & Inheritance, RBAC Built-in Roles (Owner/Contributor/Reader), and hands-on least privilege lab. |

---

## 💡 Key Design & Learning Philosophy

- **Student-Friendly Plain English:** Every topic breaks down technical jargon using real-world analogies (e.g. buying a laptop in a computer store, shipping containers on cargo ships, piggy bank burst credits, airport security vs VIP lounges).
- **Practical & Hands-On:** Every module includes click-by-click Azure Portal instructions, full terminal commands, exact shell scripts, and expected terminal outputs.
- **Cost-Optimization Focused:** Emphasizes free tier sizing (`Standard_B1s`, `LRS`, `Scale-to-Zero`, and budget alerts) so learners never face unexpected cloud charges while studying.
