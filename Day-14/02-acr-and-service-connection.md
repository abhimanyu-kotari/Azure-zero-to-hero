# Azure Container Registry (ACR) Setup and Service Connection

To implement Continuous Integration (CI), we need a centralized, private Docker registry to store and manage our container images. In Azure, this is provided by **Azure Container Registry (ACR)**.

---

## Step 1: Create Azure Container Registry (ACR)

Run the following Azure CLI commands to create a resource group and an ACR instance:

```bash
# Define variables
RESOURCE_GROUP="azure-devops-rg"
LOCATION="eastus"
ACR_NAME="abhishekazurecicd$RANDOM" # Must be globally unique, alphanumeric only

# 1. Create Resource Group
az group create --name $RESOURCE_GROUP --location $LOCATION

# 2. Create Azure Container Registry (Standard SKU)
az acr create \
  --resource-group $RESOURCE_GROUP \
  --name $ACR_NAME \
  --sku Standard \
  --admin-enabled true

# 3. Retrieve ACR Login Server and Credentials
az acr show --name $ACR_NAME --query loginServer --output tsv
az acr credential show --name $ACR_NAME
```

---

## Step 2: Configure Service Connection in Azure DevOps

To allow Azure Pipelines to build and push container images directly to ACR, configure a Docker Registry Service Connection.

### Option A: Using Azure DevOps Web Portal (Recommended)

1. Open your project in **Azure DevOps**.
2. Click **Project Settings** (bottom-left gear icon) -> **Service connections**.
3. Click **New service connection**.
4. Select **Docker Registry** and click **Next**.
5. Select **Azure Container Registry**:
   - **Authentication type:** Service Principal.
   - **Subscription:** Select your Azure Subscription.
   - **Azure Container Registry:** Select `$ACR_NAME` created above.
   - **Service connection name:** `acr-service-connection`
   - Check **Grant access permission to all pipelines**.
6. Click **Save**.

### Option B: Using Azure CLI & Service Principal

If setting up automated access programmatically:

```bash
# 1. Get ACR Resource ID
ACR_ID=$(az acr show --name $ACR_NAME --query id --output tsv)

# 2. Create Service Principal with AcrPush role
SP_NAME="sp-azure-pipelines-acr"
SP_PASSWD=$(az ad sp create-for-rbac \
  --name $SP_NAME \
  --scopes $ACR_ID \
  --role AcrPush \
  --query password \
  --output tsv)

APP_ID=$(az ad sp list --display-name $SP_NAME --query "[0].appId" --output tsv)

echo "App ID (Username): $APP_ID"
echo "Password: $SP_PASSWD"
```

In Azure DevOps Service Connections, select **Docker Hub or Others** -> Registry type **Others**, enter:
- **Docker Registry:** `https://<your-acr-name>.azurecr.io`
- **Docker ID:** `$APP_ID`
- **Docker Password:** `$SP_PASSWD`
- **Connection name:** `acr-service-connection`
