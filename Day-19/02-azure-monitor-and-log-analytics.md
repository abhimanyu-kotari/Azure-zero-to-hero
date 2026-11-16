# Azure Monitor and Log Analytics Workspace

Azure Monitor is Microsoft Azure's native, comprehensive monitoring solution that collects, analyzes, and acts on telemetry from your cloud and on-premises environments.

---

## Azure Monitor Architecture

```
[ Sources: VMs, AKS, Storage, VNet, Apps ]
                    |
                    v
          [ Azure Monitor Engine ]
         /                        \
        v                          v
  [ Metrics Store ]          [ Log Analytics Workspace ]
  (Time-Series numerical)    (Centralized structured log data)
        |                          |
        +------------+-------------+
                     |
                     v
  +------------------------------------------------------+
  | Insights: Container Insights, VM Insights, APM      |
  | Visualize: Azure Dashboards, Workbooks, Grafana      |
  | Alert: Action Groups (Email, SMS, Webhook, PagerDuty)|
  | Automate: Autoscale rules, Azure Functions           |
  +------------------------------------------------------+
```

---

## Creating a Log Analytics Workspace via Azure CLI

```bash
# Variables
RG="monitoring-demo-rg"
LOCATION="eastus"
WORKSPACE_NAME="law-azure-zero-to-hero"

# Create Resource Group
az group create --name $RG --location $LOCATION

# Create Log Analytics Workspace
az monitor log-analytics workspace create \
  --resource-group $RG \
  --workspace-name $WORKSPACE_NAME \
  --location $LOCATION \
  --sku PerGB2018
```

---

## Kusto Query Language (KQL) for DevOps

Azure Log Analytics stores data in tabular format and queries it using **Kusto Query Language (KQL)**.

### 1. Find all pods that restarted in the last 24 hours:
```kql
KubePodInventory
| where TimeGenerated > ago(24h)
| where PodRestartCount > 0
| project TimeGenerated, Namespace, Name, PodRestartCount, PodStatus
| order by PodRestartCount desc
```

### 2. Identify top 5 Virtual Machines by CPU usage:
```kql
Perf
| where TimeGenerated > ago(1h)
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by Computer
| top 5 by AvgCPU desc
```

### 3. Search for HTTP 500 errors across application logs:
```kql
AppExceptions
| where TimeGenerated > ago(6h)
| project TimeGenerated, ProblemId, ExceptionType, OuterMessage
| summarize Count = count() by ExceptionType
| render barchart
```

---

## Setting Up Metric Alerts and Action Groups

An **Alert Rule** monitors telemetry and triggers when a threshold is breached. An **Action Group** defines what happens (e.g., send SMS, email, trigger webhook to Slack/PagerDuty).

```bash
# 1. Create an Action Group
az monitor action-group create \
  --resource-group $RG \
  --name "DevOps-OnCall-Team" \
  --short-name "DevOpsAlert" \
  --action email "AdminEmail" "devops-alerts@mycompany.com"

# 2. Create a Metric Alert for high VM CPU (> 80%)
az monitor metrics alert create \
  --resource-group $RG \
  --name "High-CPU-Alert-VM" \
  --scopes "/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/$RG/providers/Microsoft.Compute/virtualMachines/<VM_NAME>" \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action "/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/$RG/providers/microsoft.insights/actionGroups/DevOps-OnCall-Team" \
  --description "Alert when VM CPU utilization exceeds 80% for 5 consecutive minutes"
```
