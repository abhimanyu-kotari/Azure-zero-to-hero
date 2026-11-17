# ☁️ Azure Cloud Deployment & Fundamentals Masterclass

<div align="center">

![Microsoft Azure](https://img.shields.io/badge/Microsoft_Azure-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu_22.04_LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes_AKS-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Bash Shell](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnu-bash&logoColor=white)
![Status](https://img.shields.io/badge/Course_Status-7_Core_Modules_Complete-brightgreen?style=for-the-badge)

<p align="center">
  <b>Comprehensive, Student-Friendly Study Notes & Hands-on Lab Tutorials</b><br>
  Synthesized directly from the masterclass: 
  <a href="https://youtu.be/M1erI5a2wxQ"><b>Microsoft AZURE in ONE VIDEO 🔥 For Beginners [HINDI] | MPrashant</b></a>
</p>

</div>

---

## 📑 Core Syllabus: [`Month8_Azure_Cloud_Deployment`](./Month8_Azure_Cloud_Deployment)

The study notes focus on the 7 foundational and deployment modules covered in the masterclass:

1. **[01. Cloud Computing & Azure Basics](./Month8_Azure_Cloud_Deployment/01_cloud_computing_and_azure_basics.md)** (`00:04:28` - `00:51:43`)
   - ⚡ Virtualization, Hypervisors (Type 1 & 2), Cloud definitions, IaaS vs PaaS vs SaaS, Azure Global Infrastructure, and Free Account setup.
2. **[02. Compute: Azure Virtual Machines](./Month8_Azure_Cloud_Deployment/02_compute_azure_virtual_machines.md)** (`00:51:43` - `01:44:39`)
   - 💻 Laptop buying analogy, VM sizing (B/D/E/F series), SSH keys, Apache install script, **SCP file transfer from local laptop**, Cloud Shell, 6-step port verification lab (`netstat -tulnp`), NSG Port 80 experiment, and data disk mounting.
3. **[03. Docker & Containerization Basics](./Month8_Azure_Cloud_Deployment/03_docker_and_containerization_basics.md)** (`03:47:55` - `04:23:32`)
   - 📦 Container concepts, Shipping container analogy, Containers vs VMs, Docker architecture, Dockerfile recipe, Docker CLI lifecycle, and why Kubernetes (AKS) orchestration is needed.
4. **[04. Azure Container Instances (ACI)](./Month8_Azure_Cloud_Deployment/04_azure_container_instances_aci.md)** (`04:23:32` - `04:29:29`)
   - 🚀 Serverless container execution, on-demand execution with zero VM management, per-second billing, public IP & custom DNS name labels, restart policies, and Azure Files volume mounting.
5. **[05. Azure Container Apps (ACA)](./Month8_Azure_Cloud_Deployment/05_azure_container_apps_aca.md)** (`04:29:29` - `04:38:36`)
   - 🏗️ Modern microservices on Azure, Scale-to-Zero cost savings ($0 when idle), Envoy Ingress with free SSL, KEDA autoscaling, revisions, traffic splitting canary releases (80/20), and ACI vs ACA vs AKS comparison.
6. **[06. Azure Blob Storage](./Month8_Azure_Cloud_Deployment/06_azure_blob_storage.md)** (`04:43:40` - `05:25:07`)
   - 🗄️ Unstructured object storage, Storage Account hierarchy, Block/Append/Page blobs, Access Tiers (Hot/Cool/Cold/Archive rehydration), Redundancy (LRS/ZRS/GRS), time-limited SAS tokens, and AzCopy.
7. **[07. Identity & Access Management (IAM) & RBAC](./Month8_Azure_Cloud_Deployment/07_identity_and_access_management_iam.md)** (`05:25:07` - `05:57:26`)
   - 🛡️ Authentication vs Authorization (Airport & Passport analogy), Microsoft Entra ID (Azure AD), Resource Hierarchy & Inheritance, RBAC Built-in Roles (Owner/Contributor/Reader), and hands-on least privilege lab.

---

## 💡 Key Design & Learning Philosophy

- **Plain English with Real-World Analogies:** Complex topics explained using everyday examples (buying a laptop, standard shipping containers, piggy bank credits, airport passports and boarding passes).
- **Practical & Hands-On:** Step-by-step Azure Portal walkthroughs, complete bash/CLI commands, exact scripts, and expected terminal outputs.
- **Cost-Optimization Focused:** Prioritizes free tier and serverless options (`Standard_B1s`, `LRS`, `Scale-to-Zero`) so learners never face unexpected cloud charges.
