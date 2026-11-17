# 17 - Infrastructure as Code: Terraform with Azure

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `07:27:57` to `07:45:25`

---

## 1. The Problem with "ClickOps" & Why IaC is Mandatory

Clicking through the Azure Portal (colloquially called **"ClickOps"**) is acceptable for first-time learning. But in professional engineering teams, it creates major problems:
1. **Human Error:** Forgetting to check an NSG port or selecting the wrong VM size can cause production outages or billing disasters.
2. **No Version History:** There is no audit trail of who modified a firewall rule or when a subnet was changed.
3. **Slow & Non-Repeatable:** Setting up identical Staging, QA, and Production environments across 3 regions takes days of clicking.

**Infrastructure as Code (IaC)** solves this:
> You define your desired cloud architecture in human-readable code files stored in Git. Cloud engines automatically read your code and provision the exact infrastructure reliably every single time.

---

## 2. What is HashiCorp Terraform?

**Terraform** is the world’s most popular open-source, multi-cloud Infrastructure as Code software tool. It uses declarative configuration written in **HCL (HashiCorp Configuration Language)**.

### The Terraform Workflow Lifecycle:

```
+--------------------------------------------------------------------------+
|                       THE TERRAFORM LIFECYCLE                            |
+--------------------------------------------------------------------------+
|                                                                          |
|  1. terraform init     --->  Downloads the 'azurerm' provider plugin.    |
|                                                                          |
|  2. terraform plan     --->  Previews changes (Adds, Modifies, Deletes)  |
|                              WITHOUT touching Azure!                     |
|                                                                          |
|  3. terraform apply    --->  Executes API calls to Azure Resource Manager|
|                              Builds the exact cloud infrastructure!      |
|                                                                          |
|  4. terraform destroy  --->  Tears down all managed resources cleanly.   |
|                                                                          |
+--------------------------------------------------------------------------+
```

---

## 3. Hands-on Lab: Provisioning Azure Infrastructure via Terraform

In this lab, we write a complete Terraform manifest to provision a Resource Group, Virtual Network, Subnet, and Public IP.

### Step 1: Create a Working Directory & Configuration File
Create a new directory on your machine and create a file named `main.tf`:

```hcl
# 1. Define Terraform settings and required Azure provider
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.80.0"
    }
  }
}

# 2. Configure the Microsoft Azure Provider
provider "azurerm" {
  features {}
}

# 3. Create a Resource Group
resource "azurerm_resource_group" "rg" {
  name     = "rg-terraform-demo"
  location = "eastus"

  tags = {
    Environment = "Dev"
    ManagedBy   = "Terraform"
  }
}

# 4. Create a Virtual Network (VNet)
resource "azurerm_virtual_network" "vnet" {
  name                = "vnet-tf-demo"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
}

# 5. Create a Subnet inside the VNet
resource "azurerm_subnet" "subnet" {
  name                 = "subnet-tf-app"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

# 6. Create a Public IP Address
resource "azurerm_public_ip" "pip" {
  name                = "pip-tf-demo"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  allocation_method   = "Static"
  sku                 = "Standard"
}

# 7. Output the allocated Public IP address to the console
output "public_ip_address" {
  value       = azurerm_public_ip.pip.ip_address
  description = "The static public IP address generated for our infrastructure."
}
```

---

### Step 2: Initialize Terraform
Open your terminal in the directory where `main.tf` is located and authenticate via Azure CLI:
```bash
# Log in to Azure
az login

# Initialize the Terraform working directory
terraform init
```
**Output:**
```
Initializing provider plugins...
- Finding hashicorp/azurerm versions matching "~> 3.80.0"...
- Installing hashicorp/azurerm v3.80.0...
Terraform has been successfully initialized!
```

---

### Step 3: Generate Execution Plan
Run `terraform plan` to see what Azure changes will occur:
```bash
terraform plan
```
Terraform scans Azure, detects that zero resources exist, and outputs:
```
Plan: 4 to add, 0 to change, 0 to destroy.
```

---

### Step 4: Apply the Configuration to Azure
```bash
terraform apply -auto-approve
```
Within 30 seconds, Terraform connects to Azure Resource Manager and provisions all four resources in the proper dependency order!
```
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:
public_ip_address = "20.120.55.84"
```

Open the Azure Portal -> search for `rg-terraform-demo`: your VNet, Subnet, and Public IP exist exactly as defined in code!

---

### Step 5: Clean Up (Tear Down)
When you are finished practicing, delete every single resource created by Terraform with one command:
```bash
terraform destroy -auto-approve
```
```
Destroy complete! Resources: 4 destroyed.
```

---

## 4. Understanding the Terraform State File (`terraform.tfstate`)

When you run `terraform apply`, Terraform creates a critical local file called `terraform.tfstate`:
- It acts as the single source of truth mapping your code to real Azure resource IDs.
- **Rule 1:** Never manually edit `terraform.tfstate`.
- **Rule 2:** In production teams, store state remotely in an **Azure Blob Storage container** with state locking (preventing two engineers from running `apply` at the same time).

```hcl
# Configuring Remote State Backend in Azure Blob Storage
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstateabhimanyu123"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

---

## 5. Troubleshooting Common Terraform Issues

| Problem | Cause | Resolution |
| :--- | :--- | :--- |
| `terraform init` fails downloading provider. | Network/proxy issue or invalid provider version. | Verify internet access and remove `.terraform` folder and `.terraform.lock.hcl`, then re-run `terraform init`. |
| `terraform apply` fails with `ResourceExists`. | Resource already created in Azure outside Terraform. | Import the existing resource into Terraform state using `terraform import <resource_type>.<name> <azure_resource_id>`. |
| State lock error: `Error acquiring the state lock`. | Previous apply crashed or another engineer is running apply. | Run `terraform force-unlock <LOCK_ID>` after verifying no active deployment is running. |

---

## 6. Summary Checklist
- [x] Infrastructure as Code (IaC) eliminates manual human errors and enables Git versioning.
- [x] Terraform uses declarative HCL to manage cloud resources across providers.
- [x] The core workflow is: `init` -> `plan` -> `apply` -> `destroy`.
- [x] `terraform.tfstate` tracks real-world cloud metadata and must be safeguarded.
- [x] Production teams store state files remotely in Azure Blob Storage with state locking.
