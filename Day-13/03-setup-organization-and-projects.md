# Setting Up Projects and Repositories in Azure DevOps

This guide outlines the hands-on process to create an Azure DevOps Organization, configure a project, initialize a Git repository, and secure it with branch policies.

---

## Step 1: Sign in and Create an Organization

1. Navigate to [https://dev.azure.com](https://dev.azure.com).
2. Sign in using your Microsoft account or Microsoft Entra ID (Azure AD).
3. If this is your first time, click **New Organization**.
4. Enter an organization name (e.g., `devops-zero-to-hero-org`).
5. Select the hosting region closest to your developers or target resources (e.g., *Central US*, *West Europe*, *South India*).
6. Complete the captcha and click **Continue**.

---

## Step 2: Create a Project

Within an organization, projects serve as security boundaries and containers for work items, repositories, pipelines, and boards.

1. On your organization homepage, click **+ New Project**.
2. Fill in the project details:
   - **Project Name:** `Azure-DevOps-Demo`
   - **Description:** `Sample project for Azure Zero to Hero course`
   - **Visibility:** 
     - **Private:** Only authorized team members can access (recommended for enterprise).
     - **Public:** Anyone on the internet can view code and pipelines (open source).
3. Expand **Advanced Options**:
   - **Version Control:** Select **Git** (recommended) or **TFVC**.
   - **Work Item Process:** 
     - **Basic:** Simplest flow (*To Do*, *Doing*, *Done*).
     - **Agile:** Includes User Stories, Features, Epics, Tasks, and Bugs.
     - **Scrum:** Includes Product Backlog Items (PBI), Impediments, and Sprint tracking.
     - **CMMI:** Formal audit trails and strict change management.
4. Click **Create**.

---

## Step 3: Initialize and Clone an Azure Repo

1. From the left navigation menu, click **Repos** -> **Files**.
2. If the repository is empty, initialize it:
   - Check **Add a README**.
   - Check **Add a .gitignore** (e.g., Python, Node, or VisualStudio).
   - Click **Initialize**.
3. To clone the repository to your local workstation:
   - Click the **Clone** button in the top right.
   - Choose **HTTPS** or **SSH**.
   - Click **Generate Git Credentials** (if using HTTPS without Git Credential Manager).
   - Copy the URL and execute on your terminal:

```bash
git clone https://dev.azure.com/<Your-Organization>/Azure-DevOps-Demo/_git/Azure-DevOps-Demo
cd Azure-DevOps-Demo
```

---

## Step 4: Configure Branch Policies (Protecting `main`)

In production environments, developers should never push code directly to the `main` branch. All changes must go through Pull Requests with mandatory checks.

1. Navigate to **Project Settings** (bottom-left gear icon).
2. Under **Repos**, select **Repositories**.
3. Select your repository, then switch to the **Policies** tab.
4. Under **Branch Policies**, select your `main` branch.
5. Configure the following rules:
   - **Require a minimum number of reviewers:** Enable and set to at least `1`.
     - Check: *Prohibit the most recent pusher from approving their own changes*.
   - **Check for linked work items:** Set to *Required*.
   - **Check for comment resolution:** Set to *Required* (all discussion items must be resolved).
   - **Build Validation:** Click **+ Add build validation**:
     - Select your CI pipeline (e.g., `VotingApp-CI`).
     - Set policy to *Required*.
     - Trigger: *Immediately when main is updated*.
6. Click **Save**. Now direct pushes to `main` are blocked:

```bash
# Attempting direct push will be rejected:
git push origin main
# Remote rejected: TF402455: Pushes to this branch are not permitted; you must use a pull request to update this branch.
```

---

## Step 5: Configure Service Connections to Azure

To allow Azure Pipelines to deploy resources to your Azure subscription (e.g., push container images to ACR or deploy to AKS), configure a Service Connection:

1. In **Project Settings**, select **Service connections** under the **Pipelines** section.
2. Click **New service connection**.
3. Select **Azure Resource Manager** and click **Next**.
4. Choose an authentication method:
   - **Workload Identity Federation (Automatic / Recommended):** Secure, secretless authentication.
   - **Service Principal (Automatic / Manual):** Uses Client ID and Client Secret.
5. Select your Azure Subscription and Resource Group.
6. Give the connection a name (e.g., `Azure-Subscription-Service-Connection`).
7. Check **Grant access permission to all pipelines**.
8. Click **Save**.
