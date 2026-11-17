# 17 - Infrastructure as Code: Terraform with Azure

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `07:27:57` to `07:45:25`

---

## 1. What is Infrastructure as Code (IaC)?

When you first learn Azure, you click buttons in the web browser (often called **"ClickOps"**). While great for learning, ClickOps is dangerous in production:
- **Human Error:** You might forget to open a port or choose the wrong VM size.
- **Not Repeatable:** If you need to replicate the exact same setup in 3 environments (Dev, Staging, Production), clicking buttons 3 times takes hours and invites mistakes.
- **No History or Tracking:** You don't know who changed an NSG rule or why.

**Infrastructure as Code (IaC)** is the practice of defining, provisioning, and managing cloud infrastructure using machine-readable definition files (code), stored and tracked in version control (like Git).

---

## 2. What is Terraform?

**Terraform** is the world's most popular open-source Infrastructure as Code tool, created by HashiCorp.

### Why DevOps Engineers Choose Terraform:
1. **Cloud-Agnostic:** You learn one syntax—**HashiCorp Configuration Language (HCL)**—and you can provision infrastructure on Azure, AWS, Google Cloud, Kubernetes, or GitHub.
2. **Declarative Syntax:** You don't tell Terraform *how* to build a server step-by-step. You simply declare the *desired end state* (e.g., *"I want an Ubuntu VM with 2 CPU cores"*), and Terraform figures out the exact API calls to make it happen.
3. **Idempotence:** Running `terraform apply` 10 times will result in the exact same infrastructure. If nothing changed in your code, Terraform does nothing!

---

## 3. The 4 Essential Terraform Commands (The Core Lifecycle)

```
+--------------------+         +--------------------+         +--------------------+
|   terraform init   |  ---->  |   terraform plan   |  ---->  |  terraform apply   |
| (Download plugins) |         | (Preview changes)  |         | (Provision cloud)  |
+--------------------+         +--------------------+         +--------------------+
                                                                        |
                                                                        v
                                                              +--------------------+
                                                              | terraform destroy  |
                                                              | (Clean up / delete)|
                                                              +--------------------+
```

1. **`terraform init`:**
   - Initializes your working directory.
   - Downloads the required cloud provider plugins (e.g., `azurerm` for Microsoft Azure).
2. **`terraform plan`:**
   - Reads your configuration files and compares them with the live state of Azure.
   - Outputs a dry-run preview: shows green `+` for resources that will be created, yellow `~` for updates, and red `-` for deletions.
3. **`terraform apply`:**
   - Executes the plan, calls the Azure APIs, and provisions the actual resources in your subscription.
4. **`terraform destroy`:**
   - Tears down and deletes every single resource created by that configuration file in one command!

---

## 4. Hands-on Walkthrough: Provisioning Azure Resources via Terraform

In the video, M Prashant writes and executes a Terraform configuration to build an Azure Resource Group and Virtual Network:

### Step 1: Install & Authenticate
1. Ensure [Terraform](https://developer.hashicorp.com/terraform/install) and the [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) are installed on your machine.
2. Open your terminal and log into Azure:
   ```bash
   az login
   ```

### Step 2: Write `main.tf`
Create an empty folder, create a file named `main.tf`, and add the following HCL code:

```hcl
# 1. Define required providers
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

# 2. Configure the Microsoft Azure Provider
provider "azurerm" {
  features {}
}

# 3. Create a Resource Group
resource "azurerm_resource_group" "my_rg" {
  name     = "rg-terraform-demo"
  location = "East US"

  tags = {
    Environment = "Development"
    ManagedBy   = "Terraform"
  }
}

# 4. Create a Virtual Network inside the Resource Group
resource "azurerm_virtual_network" "my_vnet" {
  name                = "vnet-terraform"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.my_rg.location
  resource_group_name = azurerm_resource_group.my_rg.name
}

# 5. Create a Subnet inside the VNet
resource "azurerm_subnet" "my_subnet" {
  name                 = "subnet-app"
  resource_group_name  = azurerm_resource_group.my_rg.name
  virtual_network_name = azurerm_virtual_network.my_vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}
```

### Step 3: Run the Terraform Workflow
```bash
# 1. Initialize
terraform init

# 2. Review the plan
terraform plan

# 3. Apply changes (type 'yes' when prompted)
terraform apply
```

In under 30 seconds, open the Azure Portal. You will see `rg-terraform-demo`, `vnet-terraform`, and `subnet-app` created with all your specified tags!

### Step 4: Clean Up
When you are done testing:
```bash
terraform destroy
```
Terraform deletes all resources cleanly, ensuring zero leftover costs.

---

## 5. Summary Checklist
- [x] Infrastructure as Code (IaC) replaces manual clicking with version-controlled, repeatable code.
- [x] Terraform uses declarative HCL syntax to describe desired cloud infrastructure.
- [x] Master the 4 commands: `init` (setup), `plan` (preview), `apply` (build), `destroy` (cleanup).
- [x] Always authenticate using `az login` when running Terraform locally.
