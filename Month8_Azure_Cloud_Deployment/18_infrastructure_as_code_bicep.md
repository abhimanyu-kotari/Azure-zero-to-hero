# 18 - Infrastructure as Code: Azure Bicep

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `07:45:25` to `07:54:36`

---

## 1. Why Did Microsoft Create Azure Bicep?

Before Bicep existed, the native way to write Infrastructure as Code in Azure was using **ARM Templates (Azure Resource Manager Templates)** written in raw **JSON**.

### The Problem with ARM JSON Templates:
- JSON is verbose, difficult to read, and painful to write by hand.
- Creating a simple storage account required 50+ lines of nested curly braces `{ }`, brackets `[ ]`, and escaping expressions like `"[concat('storage', uniqueString(resourceGroup().id))]"`!
- Developers and DevOps engineers hated writing raw JSON templates.

**To solve this, Microsoft developed Azure Bicep.**

---

## 2. What is Azure Bicep?

**Azure Bicep** is a modern, domain-specific language (DSL) created by Microsoft for declaratively deploying Azure resources.

Think of Bicep as a **clean, clean-syntax wrapper around ARM templates**:
- Writing Bicep code requires **50% to 70% less code** than raw JSON.
- It is human-friendly, readable, and feels natural to developers (similar to TypeScript or Python).
- Under the hood, Bicep code compiles ("transpiles") directly into standard ARM template JSON before deployment.

```
+--------------------+       Transpiles / Compiles       +--------------------+
|    main.bicep      |  ------------------------------>  |     main.json      |
| (Clean, Readable)  |         az deployment...          | (Raw ARM Template) |
+--------------------+                                   +--------------------+
                                                                    |
                                                                    v
                                                         +--------------------+
                                                         |  Azure Resource    |
                                                         |  Manager (ARM API) |
                                                         +--------------------+
```

---

## 3. Superpowers of Azure Bicep

### 1. Day-0 Support for All Azure Features
Because Bicep is maintained directly by Microsoft's Azure engineering team, **every new Azure service or API feature is supported in Bicep on Day 1**! (With third-party tools like Terraform, you often have to wait weeks for a provider release).

### 2. Zero State Files to Manage
Terraform requires maintaining a `terraform.tfstate` file (and locking it in cloud storage) to remember what it built. **Bicep has no state file.** The Azure Resource Manager itself acts as the living source of truth.

### 3. World-Class VS Code Extension
Installing the official Bicep extension in VS Code gives you auto-complete, parameter validation, snippet templates, and instant syntax error highlighting.

---

## 4. Bicep vs. Terraform: How to Choose?

| Dimension | Azure Bicep | HashiCorp Terraform |
| :--- | :--- | :--- |
| **Cloud Scope** | **Azure Only** (Native). | **Multi-Cloud** (Azure, AWS, GCP, K8s). |
| **State Management** | **No state file** (Azure ARM is state). | Requires `terraform.tfstate` file. |
| **Feature Release** | **Day-0 support** for all new Azure APIs. | Depends on provider plugin updates. |
| **Tooling** | Native to Azure CLI and Azure Portal. | Requires separate Terraform CLI binary. |
| **Best Choice When...** | Your company uses **100% Microsoft Azure** and wants simple, zero-state deployments. | Your company uses **multi-cloud** environments (e.g. Azure + AWS) and wants one unified tool. |

---

## 5. Hands-on Walkthrough: Deploying a Storage Account with Bicep

In the video, M Prashant writes and deploys a clean Bicep template:

### Step 1: Write `main.bicep`
Create a file named `main.bicep`:

```bicep
// 1. Parameters (allows customizing values during deployment)
@description('The Azure region where resources will be deployed')
param location string = resourceGroup().location

@description('Unique prefix for the storage account name')
param storagePrefix string = 'stg'

// 2. Variables (calculated values)
var uniqueStorageName = '${storagePrefix}${uniqueString(resourceGroup().id)}'

// 3. Resource Definition: Azure Storage Account
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: uniqueStorageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
    supportsHttpsTrafficOnly: true
  }
}

// 4. Outputs (returns useful information after deployment)
output storageId string = storageAccount.id
output storageEndpoint string = storageAccount.properties.primaryEndpoints.blob
```

### Step 2: Deploy Using Azure CLI
Open your terminal and run a single Azure CLI command to deploy:

```bash
# 1. Create a Resource Group
az group create --name rg-bicep-demo --location eastus

# 2. Deploy the Bicep template
az deployment group create \
  --resource-group rg-bicep-demo \
  --template-file main.bicep
```

Azure will validate the template, automatically transpile it, and provision the Storage Account in seconds!

---

## 6. Summary Checklist
- [x] Azure Bicep is Microsoft's clean, modern, native alternative to complex ARM JSON templates.
- [x] Provides Day-0 support for all Azure services with zero state file management.
- [x] Transpiles directly into standard ARM template JSON under the hood.
- [x] Choose Bicep for 100% Azure-native projects; choose Terraform for multi-cloud enterprise projects.
