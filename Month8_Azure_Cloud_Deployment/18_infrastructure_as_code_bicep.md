# 18 - Native Azure IaC: Azure Bicep

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `07:45:25` to `07:54:36`

---

## 1. Why Azure Bicep? The Problem with ARM JSON Templates

Before Bicep existed, the native way to automate Azure deployments was **ARM Templates** (Azure Resource Manager templates) written in raw JSON.

### The Pain of ARM JSON:
- **Excessive Verbosity:** Creating a simple storage account and virtual network required 300+ lines of complex JSON brackets, quotes, and commas.
- **Painful Syntax:** Expressions like `"[concat('vnet-', parameters('environment'), '-', uniqueString(resourceGroup().id))]"` were unreadable and error-prone.
- **No Modular Modularity:** Reusing components across projects was extremely difficult.

To solve this, Microsoft developed **Azure Bicep**:
> **Bicep** is a transparent, concise domain-specific language (DSL) designed specifically for Azure. It replaces bulky JSON templates with clean, human-readable code.

```
+-------------------+      bicep build      +-------------------+
|    main.bicep     |  ------------------>  |    azuredeploy.   |  ---> Sent to Azure Resource
| (25 lines, clean) |                       |    json (ARM)     |       Manager Engine
+-------------------+                       +-------------------+
```

---

## 2. Azure Bicep vs. HashiCorp Terraform

A standard enterprise design decision is choosing between Terraform and Bicep:

| Feature | Azure Bicep | HashiCorp Terraform |
| :--- | :--- | :--- |
| **Cloud Scope** | **Azure Native only** | **Multi-Cloud** (Azure, AWS, GCP, etc.) |
| **State Management** | **No State Files!** Azure platform natively tracks resource state. | Requires managing and locking `terraform.tfstate` |
| **New Azure Features** | **Day-0 Support** (Available instantly upon new Azure API release) | Minor delay waiting for `azurerm` provider updates |
| **Tooling & Setup** | Built directly into Azure CLI (`az deployment`) | Requires standalone Terraform binary download |
| **Learning Curve** | Extremely easy for Azure administrators | Moderate (HCL + State mechanics) |

---

## 3. Hands-on Lab: Deploying Infrastructure with Bicep

In this lab, we write a clean Bicep file to deploy an Azure Storage Account and Virtual Network:

### Step 1: Write `main.bicep`
Create a file named `main.bicep`:

```bicep
// 1. Define input parameters with defaults
param location string = resourceGroup().location

@description('Unique suffix for storage account name')
param storagePrefix string = 'storebicep'

// 2. Compute a globally unique name using uniqueString()
var uniqueStorageName = '${storagePrefix}${uniqueString(resourceGroup().id)}'

// 3. Deploy an Azure Storage Account
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

// 4. Deploy an Azure Virtual Network
resource virtualNetwork 'Microsoft.Network/virtualNetworks@2023-09-01' = {
  name: 'vnet-bicep-demo'
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.5.0.0/16'
      ]
    }
    subnets: [
      {
        name: 'subnet-bicep-workload'
        properties: {
          addressPrefix: '10.5.1.0/24'
        }
      }
    ]
  }
}

// 5. Output values
output deployedStorageName string = storageAccount.name
output deployedVNetId string = virtualNetwork.id
```

Notice how much cleaner and shorter this is compared to 200 lines of JSON!

---

### Step 2: Deploy Using Azure CLI
Deploy your Bicep template directly to an Azure Resource Group:

```bash
# 1. Create a Resource Group
az group create --name rg-bicep-demo --location eastus

# 2. Deploy the Bicep template
az deployment group create \
  --resource-group rg-bicep-demo \
  --template-file main.bicep
```

**Output:**
```json
{
  "properties": {
    "provisioningState": "Succeeded",
    "outputs": {
      "deployedStorageName": {
        "type": "String",
        "value": "storebicep7y3x8zq"
      }
    }
  }
}
```

---

### Step 3: Decompiling & Compiling (Optional)
To verify that Bicep is pure syntactic sugar over ARM JSON:
```bash
# Compile Bicep into native ARM JSON:
az bicep build --file main.bicep

# Decompile existing ARM JSON into clean Bicep code:
az bicep decompile --file azuredeploy.json
```

---

## 4. Troubleshooting Common Bicep Issues

| Problem | Cause | Resolution |
| :--- | :--- | :--- |
| Deployment fails with `InvalidTemplate`. | Syntax error or unsupported API version. | Check API version string (e.g., `Microsoft.Storage/storageAccounts@2023-01-01`). Use VS Code Bicep extension for real-time validation. |
| Name collision on globally unique resources. | Storage account name already taken globally. | Use `${uniqueString(resourceGroup().id)}` in your Bicep variables to guarantee uniqueness. |
| `az bicep` command not recognized. | Azure CLI Bicep component not installed. | Run `az bicep install` to install or upgrade the Bicep CLI tool. |

---

## 5. Summary Checklist
- [x] Bicep is Microsoft's concise, readable language for native Azure IaC.
- [x] Transpiles directly into native ARM JSON templates before submission to Azure.
- [x] Requires no external state files; Azure Resource Manager handles state tracking natively.
- [x] Provides Day-0 support for all new Azure services and API releases.
