# Real-Time DevOps Use Cases for Azure Functions

DevOps engineers frequently leverage Azure Functions to automate cloud governance, clean up orphaned resources, reduce cloud spend, and implement event-driven ChatOps.

---

## Use Case 1: Automated Cleanup of Orphaned Azure Disks (Cost Optimization)

When virtual machines are deleted in Azure, the associated Managed OS Disks and Data Disks are often left behind in an unattached state. These orphaned disks continue to accrue storage charges every month.

A Timer-triggered Azure Function running on a nightly CRON schedule (`0 0 0 * * *`) scans subscriptions and automatically removes unattached disks:

```python
import logging
import azure.functions as func
from azure.identity import DefaultAzureCredential
from azure.mgmt.compute import ComputeManagementClient

def main(mytimer: func.TimerRequest) -> None:
    logging.info('Timer trigger function initiated to clean orphaned Azure disks.')

    subscription_id = "<YOUR-AZURE-SUBSCRIPTION-ID>"
    credential = DefaultAzureCredential()
    compute_client = ComputeManagementClient(credential, subscription_id)

    # List all managed disks in the subscription
    disks = compute_client.disks.list()
    cleaned_count = 0

    for disk in disks:
        # Check if the disk is unattached (managed_by is None)
        if disk.managed_by is None:
            logging.warning(f"Found orphaned disk: {disk.name} in RG: {disk.id.split('/')[4]}")
            
            # Uncomment below line to execute automated deletion
            # compute_client.disks.begin_delete(disk.id.split('/')[4], disk.name)
            cleaned_count += 1

    logging.info(f"Scan complete. Found {cleaned_count} unattached disks.")
```

---

## Use Case 2: ChatOps - Forwarding Azure Alerts to Slack or Microsoft Teams

When Azure Monitor fires an alert (e.g., High CPU, Storage Throttling, AKS Node NotReady), it invokes an HTTP-triggered Azure Function via a Webhook Action Group. The function parses the alert payload and formats a rich message card into a team Slack channel:

```python
import logging
import json
import urllib.request
import azure.functions as func

SLACK_WEBHOOK_URL = "https://hooks.slack.com/services/T00/B00/XXXXX"

def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Processing Azure Monitor alert webhook.')

    try:
        req_body = req.get_json()
        alert_data = req_body.get('data', {}).get('essentials', {})
        
        alert_rule = alert_data.get('alertRule', 'Unknown Alert')
        severity = alert_data.get('severity', 'Sev3')
        target_resource = alert_data.get('alertTargetIDs', ['N/A'])[0]

        # Format Slack payload
        slack_payload = {
            "text": f":warning: *Azure Alert Triggered!*\n*Rule:* {alert_rule}\n*Severity:* {severity}\n*Resource:* {target_resource}"
        }

        # Send POST request to Slack Webhook
        req = urllib.request.Request(
            SLACK_WEBHOOK_URL,
            data=json.dumps(slack_payload).encode('utf-8'),
            headers={'Content-Type': 'application/json'}
        )
        urllib.request.urlopen(req)

        return func.HttpResponse("Alert forwarded to Slack successfully.", status_code=200)

    except Exception as e:
        logging.error(f"Error processing alert: {str(e)}")
        return func.HttpResponse(f"Error: {str(e)}", status_code=500)
```
