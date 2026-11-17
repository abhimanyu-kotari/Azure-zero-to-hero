# 01 - Cloud Computing & Azure Basics

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `00:04:28` to `00:51:43`

---

## 1. What is Virtualization?

<img width="905" height="431" alt="image" src="https://github.com/user-attachments/assets/c538fc3e-ecc5-4383-b296-ee08743f8fd8" />


Before cloud computing existed, companies had to buy physical computers (called physical servers) for every single application they wanted to run. If a company needed a web server, a database server, and an email server, they had to buy three separate physical machines. Most of the time, these machines used only 5% to 10% of their total power, leaving 90% of the hardware wasted!

**Virtualization** is the technology that solved this problem.

### How Does Virtualization Work?
Virtualization lets you divide a single physical computer into multiple independent "virtual computers." Each virtual computer is called a **Virtual Machine (VM)**. Each VM thinks it is a real computer with its own CPU, memory, hard drive, and Operating System (like Windows or Linux).

### The Hypervisor: The Brain of Virtualization
The secret behind virtualization is a piece of software called a **Hypervisor** (also known as a Virtual Machine Monitor). 

The hypervisor sits between the physical hardware and the virtual machines. It manages the physical hardware and shares it fairly among all the virtual machines.

There are two types of hypervisors:
1. **Type-1 Hypervisor (Bare-Metal):**
   - Installed directly on top of the physical computer hardware (no base OS needed).
   - **Examples:** VMware ESXi, Microsoft Hyper-V, KVM.
   - **Used in:** Enterprise datacenters and cloud providers like Microsoft Azure.
2. **Type-2 Hypervisor (Hosted):**
   - Installed on top of an existing Operating System like an everyday application.
   - **Examples:** Oracle VirtualBox, VMware Workstation.
   - **Used by:** Students and developers practicing on their laptops.

### Key Benefits of Virtualization:
- **Cost Savings:** Run multiple servers on one physical box instead of buying 10 separate physical computers.
- **Resource Efficiency:** Fully utilize 70%–80% of hardware capacity instead of wasting 90%.
- **Isolation:** If one virtual machine crashes or catches a virus, other virtual machines on the same hardware are completely unaffected.
- **Quick Setup:** You can create, clone, or delete a new virtual machine in minutes without touching physical cables.

---

## 2. What is Cloud Computing?

**Cloud Computing** is the on-demand delivery of computing services—such as servers, storage, databases, networking, and software—over the internet ("the cloud") with **pay-as-you-go pricing**.

### The Simple Electricity Analogy:
Think about how you use electricity at home:
- You don't build a power plant in your backyard.
- You don't buy turbines or maintain electrical wires.
- You simply plug your charger into the wall socket. At the end of the month, the electric company bills you only for the units of electricity you actually consumed.

**Cloud computing works the exact same way for computers:**
Instead of buying expensive servers and keeping them in an office room with cooling and backup generators, you rent computing power from a cloud provider (like Microsoft). When you are done using it, you shut it down and stop paying.

### Core Characteristics of Cloud Computing:
- **On-Demand Self-Service:** You can create servers, storage, or databases yourself in seconds with a few mouse clicks.
- **Broad Network Access:** You can access your resources from anywhere in the world using an internet connection.
- **Resource Pooling:** The cloud provider pools huge amounts of hardware and securely shares it among millions of customers.
- **Rapid Elasticity:** You can instantly scale up (add more power/servers) when users increase, and scale down when traffic drops.
- **Measured Service (Pay-As-You-Go):** You pay strictly for what you use, calculated down to the second.

---

## 3. Cloud Service Models: IaaS vs. PaaS vs. SaaS

Cloud services are categorized into three main models depending on how much work the cloud provider handles versus how much you manage:

```
+-------------------------------------------------------------------------+
|                  On-Premises       IaaS           PaaS          SaaS    |
| Applications     [ You Manage ]  [ You Manage ] [ You Manage ] [ Cloud] |
| Data             [ You Manage ]  [ You Manage ] [ You Manage ] [ Cloud] |
| Runtime          [ You Manage ]  [ You Manage ] [ Cloud      ] [ Cloud] |
| Middleware       [ You Manage ]  [ You Manage ] [ Cloud      ] [ Cloud] |
| Operating System [ You Manage ]  [ You Manage ] [ Cloud      ] [ Cloud] |
| Virtualization   [ You Manage ]  [ Cloud      ] [ Cloud      ] [ Cloud] |
| Servers          [ You Manage ]  [ Cloud      ] [ Cloud      ] [ Cloud] |
| Storage          [ You Manage ]  [ Cloud      ] [ Cloud      ] [ Cloud] |
| Networking       [ You Manage ]  [ Cloud      ] [ Cloud      ] [ Cloud] |
+-------------------------------------------------------------------------+
```

### 1. Infrastructure as a Service (IaaS)
- **Concept:** You rent the basic raw building blocks of computing: virtual machines, storage, and networks.
- **What the Cloud Provider Manages:** Physical hardware, cooling, cables, power, and the hypervisor.
- **What You Manage:** The Operating System (installing Windows/Ubuntu), security patches, software updates, and application code.
- **Azure Example:** Azure Virtual Machines (VMs), Azure Virtual Networks (VNets), Azure Disk Storage.
- **Analogy:** Renting an empty apartment. The landlord provides walls and roof, but you bring your own furniture, paint, and lock.

### 2. Platform as a Service (PaaS)
- **Concept:** The cloud provider manages the operating system, runtime, web server, and underlying software. You only bring your application code and data.
- **What the Cloud Provider Manages:** Hardware, OS, security patches, scaling, runtime environments (Python, Node.js, .NET, Java).
- **What You Manage:** Your application code and database data.
- **Azure Example:** Azure App Services, Azure Container Apps (ACA), Azure SQL Database.
- **Analogy:** Renting a fully furnished apartment. You just bring your clothes and suitcase.

### 3. Software as a Service (SaaS)
- **Concept:** A complete, ready-to-use software application delivered over the web browser. You don't worry about servers, code, or databases.
- **What the Cloud Provider Manages:** Everything (hardware, OS, code, updates, backups).
- **What You Manage:** Your user settings and documents.
- **Examples:** Microsoft 365 (Word, Excel online), Gmail, Google Drive, OneDrive.
- **Analogy:** Eating at a restaurant. You simply sit down, eat your meal, pay the bill, and leave without washing dishes.

---

## 4. Introduction to Microsoft Azure

**Microsoft Azure** is Microsoft's public cloud computing platform. It is one of the top two largest cloud providers in the world (alongside AWS), offering hundreds of services across compute, networking, storage, AI, and security.

### Azure Global Infrastructure
To deliver fast and reliable services globally, Microsoft built massive physical datacenters connected by their own private fiber-optic network:
1. **Datacenters:** Giant secure buildings filled with thousands of physical servers, high-speed networking, generators, and cooling.
2. **Regions:** A geographical area containing one or more datacenters connected through a dedicated low-latency network (e.g., *East US*, *Central India*, *West Europe*).
3. **Availability Zones (AZs):** Physically separate datacenters within the same region. Each zone has independent power, cooling, and network connections to protect against building-level failures.
4. **Region Pairs:** Every Azure region is paired with another region in the same geography (at least 300 miles away) for disaster recovery (e.g., *East US* is paired with *West US*).

### Azure Management Hierarchy
To keep your cloud resources organized, Azure uses a four-level hierarchy:
1. **Management Groups:** Manage compliance, policies, and access across multiple subscriptions.
2. **Subscriptions:** The billing and agreement boundary (where your credit card is linked and invoices are generated).
3. **Resource Groups:** A logical folder that holds related Azure resources (e.g., VMs, disks, VNets) so you can manage, monitor, and delete them together.
4. **Resources:** The actual individual services you create (e.g., a Virtual Machine, Storage Account, or Database).

---

## 5. Getting Started: Azure Free Account Setup

Microsoft provides an **Azure Free Account** for beginners and students to learn:
- **$200 USD (or equivalent local currency) Free Credit** to use within the first 30 days on any service.
- **12 Months of Popular Free Services:** Free limited hours for Linux/Windows virtual machines, storage, and SQL databases.
- **Always Free Services (55+ services):** Azure Functions (1 million requests/month free), Azure DevOps, Azure App Service (F1 free tier).

### Prerequisites to Sign Up:
1. A Microsoft account (@outlook.com, @hotmail.com, or personal email).
2. A valid phone number (for SMS identity verification).
3. A credit card or debit card (used purely for identity validation; Azure does not charge you unless you explicitly upgrade to paid tier).

### Summary Checklist
- [x] Understand that virtualization divides physical hardware into VMs via hypervisors.
- [x] Understand cloud computing as flexible, pay-as-you-go computing over the internet.
- [x] Remember the difference: IaaS (rent hardware & OS), PaaS (rent platform, bring code), SaaS (ready-to-use software).
- [x] Know Azure's hierarchy: Subscriptions -> Resource Groups -> Resources.
