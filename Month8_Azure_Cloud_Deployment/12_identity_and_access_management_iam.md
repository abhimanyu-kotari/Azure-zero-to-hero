# 12 - Identity and Access Management (IAM) & RBAC

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `05:25:07` to `05:57:26`

---

## 1. What is Identity and Access Management (IAM)?

In any organization, security is paramount. You cannot give every employee the keys to the entire company. A junior developer should not be able to delete the production database, and a financial auditor should only be able to view billing reports without touching application servers.

**Identity and Access Management (IAM)** is the security framework that ensures the right people have the right access to the right cloud resources at the right time.

---

## 2. The Golden Rule: Authentication vs. Authorization

Before diving into cloud security, every student must understand the fundamental difference between these two terms:

```
+-----------------------------------------------------------------------------------+
|  1. AUTHENTICATION (AuthN)            |  2. AUTHORIZATION (AuthZ)                 |
|  Question: "Who are you?"             |  Question: "What are you allowed to do?"  |
|                                       |                                           |
|  Verifies your identity:              |  Checks your permissions:                 |
|  - Username & Password                |  - Can you delete this VM?                |
|  - OTP / SMS code                     |  - Can you view this database?            |
|  - Multi-Factor Authentication (MFA)  |  - Can you download this file?            |
|  - Biometric fingerprint / FaceID     |                                           |
+-----------------------------------------------------------------------------------+
```

### The Airport Boarding Analogy:
- **Authentication:** You show your passport at the airport gate. The officer verifies your face matches the photo. You have proven **who you are**.
- **Authorization:** You hand over your boarding pass. If your ticket says "Economy Class," the flight attendant will not let you enter the "First Class" lounge or cockpit. The ticket defines **what you are allowed to access**.

---

## 3. What is Microsoft Entra ID (Formerly Azure Active Directory)?

**Microsoft Entra ID** is Microsoft's cloud-based identity and access management service. It handles the **Authentication** side of Azure.

When you log into the Azure Portal, Microsoft 365, or enterprise cloud applications, Entra ID is the service that authenticates your username, checks your password, enforces Multi-Factor Authentication (MFA), and verifies your account status.

### What Entra ID Manages:
- **Users:** Individual identities (e.g., `rahul@mycompany.com`).
- **Groups:** Collections of users (e.g., `DevOps-Team`, `Interns-2026`). Giving permissions to a group automatically applies to all members inside it.
- **Service Principals:** Identities created for automated applications, scripts, or CI/CD pipelines (like GitHub Actions or Azure DevOps) so they can deploy resources without using a human's credentials.
- **Managed Identities:** A special feature where Azure automatically assigns an identity to a cloud resource (like an Azure VM or Azure Function), eliminating the need for developers to write database passwords or API keys inside their code!

---

## 4. What is Azure Role-Based Access Control (RBAC)?

Once Entra ID proves *who you are*, Azure needs to know *what you can do*. This is handled by **Role-Based Access Control (RBAC)**.

Azure RBAC is an authorization system built on the **Principle of Least Privilege**:
> **Principle of Least Privilege:** Users should only be granted the minimum permissions strictly necessary to accomplish their assigned job duties, and nothing more.

---

## 5. The Three Pillars of an RBAC Assignment

An RBAC permission is created by binding three elements together:

```
+------------------+         +-------------------+         +---------------+
|     1. WHO       |         |      2. WHAT      |         |   3. WHERE    |
| Security Principal |   +   |  Role Definition  |   +   |     Scope     |
| (User, Group, SP) |         | (Owner, Reader...) |         |  (Hierarchy)  |
+------------------+         +-------------------+         +---------------+
                                       |
                                       v
                             [ ROLE ASSIGNMENT ]
```

### 1. Security Principal (WHO)
The object requesting access:
- A **User** (e.g., John)
- A **Group** (e.g., QA Team)
- A **Service Principal** (e.g., Deployment script)
- A **Managed Identity** (e.g., Azure Web App)

### 2. Role Definition (WHAT)
A collection of permissions that define what actions can or cannot be performed.

Azure provides hundreds of built-in roles, but three core roles apply everywhere:
1. **Owner:**
   - Full access to manage all resources (create, read, update, delete).
   - **Can grant and revoke access permissions to other users.**
2. **Contributor:**
   - Full access to create, update, manage, and delete all resources.
   - **CANNOT grant access to other users** (cannot modify RBAC assignments).
3. **Reader:**
   - Can only view existing resources and telemetry.
   - Cannot create, change, restart, or delete anything.

There are also **Specialized Roles** for specific services:
- *Virtual Machine Contributor:* Can create and restart VMs, but cannot touch virtual networks or storage.
- *Storage Blob Data Reader:* Allows reading files inside Blob containers without having admin access to the storage account itself.

### 3. Scope (WHERE)
The boundary of resources where the access applies. Azure organizes resources into a strict hierarchy:

```
1. Management Groups  (Organization-wide governance)
         |
         v
2. Subscriptions      (Billing boundary)
         |
         v
3. Resource Groups    (Project / department folder)
         |
         v
4. Resources          (A specific VM, Storage Account, or Database)
```

---

## 6. Scope Inheritance: How Permissions Flow Downward

**Crucial Rule for Cloud Engineers:**
Permissions assigned at a higher parent level are **automatically inherited** by every child level beneath it!

```
                    [ Subscription Scope: Contributor ]
                                    |
          +-------------------------+-------------------------+
          |                                                   |
          v                                                   v
[ Resource Group A ]                                [ Resource Group B ]
(Automatically Contributor)                         (Automatically Contributor)
          |                                                   |
          v                                                   v
   [ Virtual Machine 1 ]                               [ Storage Account 1 ]
   (Can edit/delete)                                   (Can edit/delete)
```

### Why Scope Matters:
- If you assign someone the **Owner** role at the **Subscription** level, they have full control over every single resource in that subscription.
- If you assign someone **Contributor** role on a specific **Resource Group** (`rg-testing`), they can create and delete servers inside that folder, but they cannot see or touch anything inside `rg-production`!
- **Best Practice:** Always grant access at the lowest feasible scope (usually the Resource Group level).

---

## 7. Hands-on Walkthrough: Managing Permissions in Azure Portal

In the video, M Prashant demonstrates how to assign permissions using the Azure Portal:

### Step 1: Open Access Control (IAM)
1. Open the [Azure Portal](https://portal.azure.com).
2. Navigate to your target resource or resource group (e.g., `rg-compute-demo`).
3. In the left navigation menu, click **Access control (IAM)**.
4. Click on the **Role assignments** tab to see who currently has permissions on this folder.

### Step 2: Add a Role Assignment
1. Click **+ Add** -> **Add role assignment**.
2. **Role Tab:** Select **Reader** (or *Virtual Machine Contributor*). Click **Next**.
3. **Members Tab:**
   - Under *Assign access to*, select **User, group, or service principal**.
   - Click **+ Select members**.
   - Type the email address of the team member or classmate you want to invite.
   - Click **Select**, then click **Next**.
4. **Review + Assign:**
   - Azure displays a summary showing: *WHO* (user), *WHAT* (Reader role), and *WHERE* (rg-compute-demo).
   - Click **Review + assign**.

### Testing the Role:
When that user logs into their Azure Portal:
- They can navigate to `rg-compute-demo` and see the running Virtual Machine.
- However, if they try to click **Delete** or **Restart**, Azure will immediately display an error:
  `AuthorizationFailed: The client does not have authorization to perform action 'Microsoft.Compute/virtualMachines/delete' over scope.`

This proves the RBAC security policy is actively protecting your cloud infrastructure!

---

## 8. Summary Checklist
- [x] Authentication (AuthN) proves your identity; Authorization (AuthZ) verifies your permissions.
- [x] Microsoft Entra ID is Azure's cloud-native identity provider for users and applications.
- [x] Azure RBAC pairs a Security Principal (who) with a Role Definition (what) at a Scope (where).
- [x] Know the three core roles: Owner (full + can grant access), Contributor (full - cannot grant access), Reader (view only).
- [x] Scope permissions inherit downwards; always practice the Principle of Least Privilege.
