# Managed Prometheus, Grafana, and Azure Network Watcher

## Azure Managed Prometheus & Managed Grafana on AKS

For Kubernetes deployments, Prometheus and Grafana are the industry standard for metrics scraping and dashboard visualization. Azure offers fully managed, scalable versions of both without needing to manage Prometheus servers or Grafana storage.

### Enabling Managed Prometheus on an Existing AKS Cluster:

```bash
# Variables
RG="monitoring-demo-rg"
CLUSTER="aks-cluster-demo"
GRAFANA_NAME="grafana-devops-$RANDOM"

# 1. Create Azure Managed Grafana
az grafana create \
  --name $GRAFANA_NAME \
  --resource-group $RG

# 2. Enable Azure Managed Prometheus on AKS and link with Grafana
az aks update \
  --name $CLUSTER \
  --resource-group $RG \
  --enable-azure-monitor-metrics \
  --azure-monitor-workspace-resource-id "/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/$RG/providers/microsoft.monitor/accounts/amw-$CLUSTER" \
  --grafana-resource-id "/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/$RG/providers/Microsoft.Dashboard/grafana/$GRAFANA_NAME"
```

Once linked, Grafana automatically provisions standard Kubernetes dashboards:
- **Kubernetes / Compute Resources / Cluster**
- **Kubernetes / Compute Resources / Namespace (Pods)**
- **Kubernetes / Compute Resources / Node**

---

## Azure Network Watcher for Network Diagnostics

Azure Network Watcher provides tools to monitor, diagnose, view network metrics, and enable or disable logs for resources in an Azure Virtual Network.

### Key Capabilities for DevOps Engineers:

1. **IP Flow Verify:**
   - Verifies if a packet is allowed or denied to/from a virtual machine based on configured Network Security Groups (NSGs).
   - Indispensable for debugging "why can't my app reach the database?" questions:

```bash
az network watcher test-ip-flow \
  --resource-group $RG \
  --vm <VM_NAME> \
  --direction Inbound \
  --protocol TCP \
  --local 80 \
  --remote 203.0.113.12:54321
```

2. **Next Hop:**
   - Determines where traffic from a VM is being routed (e.g., VirtualNetwork, Internet, VirtualAppliance/Firewall, or None). Useful for verifying route tables.

```bash
az network watcher show-next-hop \
  --resource-group $RG \
  --vm <VM_NAME> \
  --source-ip 10.0.1.4 \
  --dest-ip 8.8.8.8
```

3. **NSG Flow Logs & Traffic Analytics:**
   - Logs information about IP traffic flowing through an NSG. Sent to Azure Storage and visualized with Traffic Analytics to detect security vulnerabilities, anomalous traffic spikes, or unexpected outbound connections.

4. **Packet Capture:**
   - Initiates an on-demand packet capture (`.cap` file) directly on a VM without logging in via SSH, enabling Wireshark analysis for complex TCP handshake or TLS negotiation failures:

```bash
az network watcher packet-capture create \
  --resource-group $RG \
  --vm <VM_NAME> \
  --name "debug-db-traffic" \
  --storage-account <STORAGE_NAME>
```
