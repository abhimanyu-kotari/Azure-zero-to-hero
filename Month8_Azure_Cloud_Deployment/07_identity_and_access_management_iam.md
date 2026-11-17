# 🛡️ 07 - Security & Governance: Microsoft Entra ID & Azure RBAC

<div align="center">

![Microsoft Entra ID](https://img.shields.io/badge/Microsoft_Entra_ID-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Azure RBAC](https://img.shields.io/badge/Azure_RBAC-Least_Privilege-critical?style=for-the-badge)
![Built-in Roles](https://img.shields.io/badge/Roles-Owner_%7C_Contributor_%7C_Reader-blue?style=for-the-badge)
![Security](https://img.shields.io/badge/Security-AuthN_vs_AuthZ-purple?style=for-the-badge)

</div>

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `05:25:07` to `05:57:26` (Identity, Governance & Access Control)

---

## 🛂 1. Authentication vs. Authorization (The Airport Analogy)

In cloud security and identity management, engineers often confuse these two fundamental security concepts. In the video, M Prashant explains the distinction using an everyday airport travel analogy:

```
+---------------------------------------------------------------------------------------------------+
|                                 AUTHENTICATION VS. AUTHORIZATION                                  |
+---------------------------------------------------------------------------------------------------+
|                                                                                                   |
|   AUTHENTICATION (AuthN)                              AUTHORIZATION (AuthZ)                       |
|   "Who are you?"                                      "What are you allowed to do?"               |
|                                                                                                   |
|   - Verifying your true identity                      - Determining permissions & privileges      |
|   - Username & Password                               - Role-Based Access Control (RBAC)          |
|   - Multi-Factor Authentication (MFA)                 - Can this person start or stop a VM?       |
|   - Microsoft Authenticator push notification         - Can this person delete a database?        |
|                                                                                                   |
|   AIRPORT ANALOGY:                                    AIRPORT ANALOGY:                            |
|   Showing your Passport & ID to airport security      Your Boarding Pass allowing you onto Flight |
|   to prove that you are really who you claim to be!   or into the First-Class VIP Lounge!         |
|                                                                                                   |
+---------------------------------------------------------------------------------------------------+
```

---

## 2. What is Microsoft Entra ID (formerly Azure Active Directory)?

**Microsoft Entra ID** is Microsoft's enterprise cloud-based identity and access management service. It manages user logins, organization directory memberships, multi-factor authentication, and single sign-on (SSO).

### The Four Core Identity Types in Entra ID:
1. **Users:** Human beings (employees, developers, contractors, or interns) with an email address and password.
2. **Groups:** Collections of users (e.g., `DevOps-Engineers`, `Finance-Auditors`). Assigning an RBAC permission to a group automatically grants it to every member inside the group!
3. **Service Principals:** Identities created for automated applications, CI/CD pipelines, or deployment scripts (e.g., GitHub Actions deploying code into Azure).
4. **Managed Identities:** Eliminates the need for developers to manage credentials in code. An Azure VM can authenticate directly to Azure Key Vault or Blob Storage using its own identity without hardcoding passwords!

---

## 3. Azure Resource Hierarchy & Scope

Azure organizes all cloud resources into a strict hierarchical tree:

```
+-------------------------------------------------------------------------+
|                        AZURE RESOURCE HIERARCHY                         |
+-------------------------------------------------------------------------+
|                                                                         |
|                       [ Root Management Group ]                         |
|                                   |                                     |
|                                   v                                     |
|                       [ Management Groups ]                             |
|                        (Enterprise Governance)                          |
|                                   |                                     |
|                                   v                                     |
|                       [ Azure Subscriptions ]                           |
|                       (Billing & Quota Boundary)                        |
|                                   |                                     |
|                                   v                                     |
|                       [ Resource Groups ]                               |
|                       (Logical Life-cycle Containers)                   |
|                                   |                                     |
|                                   v                                     |
|                       [ Individual Resources ]                          |
|                       (VMs, Storage Accounts, VNets)                    |
|                                                                         |
+-------------------------------------------------------------------------+
```

### The Rule of Inheritance:
> **Permissions granted at a higher parent level are automatically inherited by all child resources below it.**  
> - If you assign a user the **Reader** role at the **Subscription** level, they automatically have read access to every Resource Group and every single Virtual Machine inside that entire subscription!  
> - If you assign a role at a specific **Resource Group** level, the user's permissions are strictly confined to that single group.

---

## 4. Role-Based Access Control (RBAC) & Principle of Least Privilege

**Azure RBAC** enforces the **Principle of Least Privilege**: users should only be given the absolute minimum permissions necessary to complete their job, and nothing more.

### The Three Fundamental Built-in Roles:

| Built-In Role | Can View Resources? | Can Create / Edit / Delete? | Can Grant Access to Other Users? |
| :--- | :--- | :--- | :--- |
| **Reader** | **YES** | NO | NO |
| **Contributor** | **YES** | **YES** | **NO** (Cannot modify RBAC permissions) |
| **Owner** | **YES** | **YES** | **YES** (Full administrative rights) |

> [!WARNING]
> Never make junior developers or interns an **Owner** or **Contributor** at the Subscription level! A single misconfigured script or accidental click could delete production databases.

### Specialized Built-in Roles:
- **Virtual Machine Contributor:** Can create, stop, restart, and delete VMs, but cannot modify virtual networks or storage accounts outside their scope.
- **Storage Blob Data Contributor:** Allows read, write, and delete operations on Blob containers and files via Entra ID authentication.
- **User Access Administrator:** Can manage user permissions and role assignments, but cannot create VMs or storage.

---

## 5. Hands-on Lab: Creating a User & Assigning RBAC Roles

In this lab from the video, M Prashant demonstrates creating a new user, proving they have zero initial access, granting the `Reader` role, and observing the exact error when attempting an unauthorized action:

### Step 1: Create a New User in Microsoft Entra ID
1. In the Azure Portal search bar, type **Microsoft Entra ID**.
2. In the left navigation menu, click **Users** $\rightarrow$ click **+ New user** $\rightarrow$ select **Create new user**.
3. **Identity Details:**
   - **User principal name:** `student.intern@<yourtenant>.onmicrosoft.com`
   - **Display name:** `Student Intern`
   - **Password:** Select *Auto-generate password* (or set a temporary password and copy it).
4. Click **Review + create** $\rightarrow$ click **Create**.

---

### Step 2: Test Access Without Role Assignment
1. Open a new **Private / Incognito browser window**.
2. Navigate to `https://portal.azure.com` and log in with `student.intern`.
3. Search for **Virtual Machines**.
   - **Result:** You see **no resources** and cannot create anything. Because no RBAC role has been assigned, the user has zero visibility and zero permissions!

---

### Step 3: Assign the "Reader" Role at Resource Group Scope
1. Switch back to your administrator account browser session.
2. Navigate to the Resource Group `azure-vm-rg`.
3. In the left menu, click **Access control (IAM)**.
4. Click **+ Add** $\rightarrow$ select **Add role assignment**.
5. **Role Tab:** Select **Reader** $\rightarrow$ click **Next**.
6. **Members Tab:**
   - Assign access to: **User, group, or service principal**.
   - Click **+ Select members** $\rightarrow$ search for `Student Intern`.
7. Click **Review + assign**.

---

### Step 4: Verify Reader Permissions & Observe Unauthorized Error
1. Switch back to the `Student Intern` Incognito browser session and refresh the page.
2. Open `azure-vm-rg`:
   - **Result:** The user can now see the VM `mywebserver`, view its Public IP, CPU graphs, and networking rules!
3. Now attempt to click **Restart** or **Stop**:
   - **Result:** Azure displays an immediate error notification:  
     `The client 'student.intern@...' with object id '...' does not have authorization to perform action 'Microsoft.Compute/virtualMachines/restart/action'.`
   - The user is strictly confined to read-only visibility!

---

### Step 5: Elevate to "Contributor" Role
1. In your Admin window, go back to **Access control (IAM)** $\rightarrow$ **Role assignments**.
2. Add a new role assignment for `Student Intern`: select **Contributor**.
3. Now the intern can start, stop, edit, and manage resources, but **cannot** delete permissions or grant access to other team members!

---

## 6. Troubleshooting Common RBAC & Identity Issues

| Problem | Root Cause | Solution |
| :--- | :--- | :--- |
| User cannot see resources after role assignment. | Entra ID token caching delay. | Have the user log out and log back in, or wait 2 minutes for the OAuth token to refresh. |
| User cannot assign roles to colleagues despite being Contributor. | Contributor role explicitly denies authorization actions. | Only **Owner** or **User Access Administrator** can grant RBAC permissions to other users. |
| Service principal deployment fails with `Forbidden (403)`. | Pipeline identity lacks permissions on target scope. | Grant the service principal the `Contributor` role at the target Resource Group level. |

---

## 7. Summary Checklist
- [x] Authentication (AuthN) proves *identity*; Authorization (AuthZ) enforces *permissions*.
- [x] Microsoft Entra ID is the central cloud identity provider for Microsoft Azure.
- [x] Azure RBAC enforces the Principle of Least Privilege across Subscriptions and Resource Groups.
- [x] Permissions cascade downwards through inheritance from parent scopes to child resources.
- [x] The primary roles are Owner (Full control + Delegation), Contributor (Full resource control, No delegation), and Reader (Read-only).
