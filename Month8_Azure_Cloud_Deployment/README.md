# ☁️ Month 8: Azure Cloud Deployment & Fundamentals Masterclass

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

## 🗺️ Visual Architecture Roadmap: The 7 Core Modules

```
                                  🌐 [ INTERNET VISITORS ]
                                             │
                       ┌─────────────────────┴─────────────────────┐
                       ▼                                           ▼
          ┌─────────────────────────┐                 ┌─────────────────────────┐
          │  Module 02: Azure VM    │                 │  Module 05: Azure ACA   │
          │  Apache Web Server      │                 │  Serverless Microservice│
          │  Port 80 / NSG Rules    │                 │  Scale-to-Zero ($0)     │
          └────────────┬────────────┘                 └────────────┬────────────┘
                       │                                           │
                       ├─────────────────────┬─────────────────────┤
                       ▼                     ▼                     ▼
          ┌─────────────────────────┐ ┌──────────────┐ ┌─────────────────────────┐
          │  Module 06: Azure Blob  │ │Module 03:    │ │  Module 04: Azure ACI   │
          │  Storage Account        │ │Docker & K8s  │ │  Serverless Container   │
          │  SAS Tokens / Tiers     │ │Architecture  │ │  Fast 5-second deploy   │
          └────────────┬────────────┘ └──────────────┘ └─────────────────────────┘
                       │
                       ▼
          ┌─────────────────────────────────────────────────────────────────────┐
          │  Module 07: Microsoft Entra ID (IAM) & RBAC                         │
          │  Least Privilege Security: Owner 👑 | Contributor 🛠️ | Reader 👁️      │
          └─────────────────────────────────────────────────────────────────────┘
```

---

## 📑 Core Syllabus Index & Timelines

| Module | Topic File | Video Timeframe | Status | Core Concepts & Hands-on Labs Covered |
| :---: | :--- | :---: | :---: | :--- |
| **01** | [**01_cloud_computing_and_azure_basics.md**](./01_cloud_computing_and_azure_basics.md) | `00:04:28` - `00:51:43` | 🟢 Active | ⚡ Virtualization, Hypervisors (Type 1 & 2), Cloud definitions, IaaS vs PaaS vs SaaS, Azure Global Infrastructure, and Free Account setup. |
| **02** | [**02_compute_azure_virtual_machines.md**](./02_compute_azure_virtual_machines.md) | `00:51:43` - `01:44:39` | 🟢 Active | 💻 Laptop buying analogy, VM Sizing (`Standard_B1s`), SSH keys, Apache script, **local SCP file upload**, Cloud Shell, 6-step port verification lab (`netstat -tulnp`), and NSG port 80 rule. |
| **03** | [**03_docker_and_containerization_basics.md**](./03_docker_and_containerization_basics.md) | `03:47:55` - `04:23:32` | 🟢 Active | 📦 Shipping container analogy, Containers vs VMs, Dockerfile anatomy, Docker CLI commands, and why Kubernetes (AKS) orchestration is needed. |
| **04** | [**04_azure_container_instances_aci.md**](./04_azure_container_instances_aci.md) | `04:23:32` - `04:29:29` | 🟢 Active | 🚀 Pure serverless containers, zero VM management, per-second billing, restart policies (Always, Never, OnFailure), custom DNS labels, and Azure Files mounts. |
| **05** | [**05_azure_container_apps_aca.md**](./05_azure_container_apps_aca.md) | `04:29:29` - `04:38:36` | 🟢 Active | 🏗️ Modern microservices on Azure, Envoy Ingress with free auto-renewing SSL, KEDA autoscaling, **Scale-to-Zero ($0 idle cost)**, and canary traffic splitting (80/20). |
| **06** | [**06_azure_blob_storage.md**](./06_azure_blob_storage.md) | `04:43:40` - `05:25:07` | 🟢 Active | 🗄️ Unstructured object storage, Block/Append/Page blobs, Access Tiers (Hot, Cool, Cold, Archive), Redundancy (LRS/ZRS/GRS), time-limited SAS tokens, and AzCopy. |
| **07** | [**07_identity_and_access_management_iam.md**](./07_identity_and_access_management_iam.md) | `05:25:07` - `05:57:26` | 🟢 Active | 🛡️ AuthN vs AuthZ (Airport & Passport analogy), Microsoft Entra ID (Azure AD), Resource Hierarchy & Inheritance, RBAC roles (Owner/Contributor/Reader), and least privilege lab. |

---

## 🎯 Key Learning Principles

> [!TIP]
> **Plain English with Real-World Analogies:**  
> Every topic breaks down complex cloud jargon into everyday mental models:
> - **Virtual Machines:** Buying a laptop at an electronics shop.
> - **Cloud Services:** An electric power grid or renting an apartment.
> - **Containers:** Standard steel shipping containers on global cargo vessels.
> - **Burstable VM Sizing:** Breaking open a piggy bank for CPU burst credits.
> - **Identity Management:** Showing a passport at airport security vs. holding a VIP lounge boarding pass.

> [!IMPORTANT]
> **Cost-Optimization Focused:**  
> All exercises are designed strictly around free tier and serverless resources (`Standard_B1s`, `LRS`, `Scale-to-Zero`, and budget alerts) so students never face unexpected charges while learning.

---

## 🛠️ Quick Command Reference

```bash
# Connect to your Azure VM via SSH
ssh -i ~/Downloads/mywebserver_key.pem azureuser@<VM_PUBLIC_IP>

# Securely copy local files to your Azure VM via SCP
scp -i ~/Downloads/mywebserver_key.pem index.html azureuser@<VM_PUBLIC_IP>:~

# Inspect active web listening ports on the server
sudo netstat -tulnp | grep :80

# Check live real-time visitor traffic
sudo tail -f /var/log/apache2/access.log
```
