# 12 - Security & Governance: Microsoft Entra ID & Azure RBAC

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `05:25:07` to `05:57:26`

---

## 1. Authentication vs. Authorization

In cloud security and identity management, people often confuse these two fundamental concepts:

```
+--------------------------------------------------------------------------+
|                  AUTHENTICATION VS. AUTHORIZATION                        |
+--------------------------------------------------------------------------+
|                                                                          |
|   AUTHENTICATION (AuthN)                  AUTHORIZATION (AuthZ)          |
|   "Who are you?"                          "What are you allowed to do?"  |
|                                                                          |
|   - Verifying your identity               - Determining permissions      |
|   - Username & Password                   - Role-Based Access Control    |
|   - Multi-Factor Authentication (MFA)     - Can user create a VM?        |
|   - Microsoft Authenticator push          - Can user delete a database?  |
|                                                                          |
|   Example: Showing your passport          Example: Being allowed inside  |
|   at airport security.                    the VIP lounge with a ticket.  |
|                                                                          |
+--------------------------------------------------------------------------+
```

---

## 2. What is Microsoft Entra ID (formerly Azure Active Directory)?

**Microsoft Entra ID** is Microsoft’s cloud-based identity and access management service. It manages user logins, organization directory memberships, multi-factor authentication, and single sign-on (SSO).

### Core Entities in Entra ID:
1. **Users:** Employees, developers, contractors, or external guests with an email and credential.
2. **Groups:** Collections of users (e.g., `DevOps-Team`, `Billing-Admins`). Assigning permissions to a group automatically applies to all members!
3. **Service Principals:** Identities created for automated applications, CI/CD pipelines, or scripts (e.g., GitHub Actions deploying to Azure).
4. **Managed Identities:** Eliminates the need for developers to manage credentials in code. An Azure VM can authenticate directly to Azure Key Vault or Blob Storage using its own identity!

---

## 3. Azure Resource Hierarchy & Scope

Azure organizes all cloud assets in a strict hierarchical tree. Every resource belongs to a Resource Group, which belongs to a Subscription, which belongs to a Management Group:

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
> **Permissions granted at a higher level are automatically inherited by all child resources below it.**  
> If you assign a user the **Reader** role at the **Subscription** level, they automatically have read access to every Resource Group and every Virtual Machine inside that entire subscription!

---

## 4. Role-Based Access Control (RBAC)

**Azure RBAC** enforces the **Principle of Least Privilege**: users should only be given the absolute minimum permissions necessary to perform their daily duties.

### The Three Fundamental Built-in Roles:

| Built-In Role | Can View Resources? | Can Create / Edit / Delete? | Can Grant Access to Other Users? |
| :--- | :--- | :--- | :--- |
| **Reader** | **YES** | NO | NO |
| **Contributor** | **YES** | **YES** | **NO** (Cannot modify RBAC permissions) |
| **Owner** | **YES** | **YES** | **YES** (Full administrative rights) |

### Specialized Built-in Roles:
- **User Access Administrator:** Can manage user permissions and role assignments, but cannot create VMs or storage.
- **Virtual Machine Contributor:** Can create, stop, restart, and delete VMs, but cannot manage virtual networks or storage accounts outside their scope.
- **Storage Blob Data Contributor:** Allows read, write, and delete operations on Blob containers and files via Entra ID authentication.

---

## 5. Hands-on Lab: Creating a User & Assigning RBAC Roles

### Step 1: Create a New User in Microsoft Entra ID
1. In the Azure Portal search bar, type **Microsoft Entra ID**.
2. In the left navigation menu, click **Users** -> **+ New user** -> **Create new user**.
3. **Identity:**
   - **User principal name:** `student.intern@<yourtenant>.onmicrosoft.com`
   - **Display name:** `Student Intern`
   - **Password:** Select *Auto-generate* or set an initial password.
4. Click **Review + create** -> **Create**.

---

### Step 2: Test Access Without Role Assignment
1. Open a new **Private / Incognito browser window**.
2. Navigate to `https://portal.azure.com` and log in with the new user credentials.
3. Search for **Virtual Machines**.
   - **Result:** You see **no resources** and cannot create anything. Because no RBAC role has been assigned, the user has zero permissions across your subscription.

---

### Step 3: Assign the "Reader" Role at Resource Group Scope
1. Switch back to your administrator account browser session.
2. Navigate to the Resource Group `rg-compute-demo`.
3. In the left menu, click **Access control (IAM)**.
4. Click **+ Add** -> **Add role assignment**.
5. **Role Tab:** Select **Reader** -> click **Next**.
6. **Members Tab:**
   - Assign access to: **User, group, or service principal**.
   - Click **+ Select members** -> search for `Student Intern`.
7. Click **Review + assign**.

---

### Step 4: Verify Reader Permissions
1. Switch back to the `Student Intern` Incognito browser session and refresh the page.
2. Open `rg-compute-demo`:
   - **Result:** The user can now see the VM `vm-web-demo` and view its Public IP, CPU graphs, and networking rules!
3. Now attempt to click **Restart** or **Stop**:
   - **Result:** Azure displays an error:  
     `The client 'student.intern@...' with object id '...' does not have authorization to perform action 'Microsoft.Compute/virtualMachines/restart/action'.`
   - The user is strictly confined to read-only visibility!

---

### Step 5: Elevate to "Contributor" Role
1. In your Admin window, go back to **Access control (IAM)** -> **Role assignments**.
2. Add a new role assignment for `Student Intern`: select **Contributor**.
3. Now the intern can start, stop, edit, and create resources, but **cannot** delete permissions or grant access to other team members.

---

## 6. Summary Checklist
- [x] Authentication (AuthN) proves *identity*; Authorization (AuthZ) enforces *permissions*.
- [x] Microsoft Entra ID is the central cloud identity provider for Microsoft Azure.
- [x] Azure RBAC enforces the Principle of Least Privilege across Management Groups, Subscriptions, and Resource Groups.
- [x] Permissions cascade downwards through inheritance from parent scopes to child resources.
- [x] The primary roles are Owner (Full control + Delegation), Contributor (Full resource control, No delegation), and Reader (Read-only).
