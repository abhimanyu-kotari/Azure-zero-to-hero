# 16 - Governance & FinOps: Azure Tags, Billing & Cost Management

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `07:19:01` to `07:27:57`

---

## 1. Why Cloud Financial Operations (FinOps) Matters

In traditional on-premises IT, buying hardware requires months of budgeting, purchase orders, and approvals (CapEx - Capital Expenditure). In the cloud, any developer can spin up a $5,000/month GPU cluster in 30 seconds with a mouse click (OpEx - Operational Expenditure).

Without strict governance and cost controls, companies frequently experience "bill shock". Azure provides built-in tools to tag, monitor, forecast, and constrain spending.

---

## 2. What are Azure Tags?

**Azure Tags** are metadata key-value pairs applied to resources, resource groups, and subscriptions. They categorize assets for billing, security, and lifecycle management.

```
+--------------------------------------------------------------------------+
|                         SAMPLE RESOURCE TAG MATRIX                       |
+--------------------------------------------------------------------------+
|  Tag Key          | Sample Tag Value      | Why it is critical           |
|  -----------------+-----------------------+----------------------------- |
|  Environment      | Production / Dev / QA | Separate test bills from prod|
|  CostCenter       | CC-4109 (Finance)     | Chargeback to departments    |
|  Owner            | user@company.com      | Contact when VM left running |
|  AutoShutdown     | True                  | Script automatically stops VM|
+--------------------------------------------------------------------------+
```

### Applying Tags:
Tags can be applied via:
1. **Azure Portal:** On any resource page under **Tags**.
2. **Azure CLI:**
   ```bash
   az tag create --resource-id <RESOURCE_ID> --tags Environment=Production Department=Engineering
   ```
3. **Azure Policy:** Enforce rules such as: *"Deny creation of any resource that lacks an `Owner` tag."*

---

## 3. Cost Analysis & Reporting

The **Cost Management + Billing** console in Azure allows you to dissect cloud spending:

### Key Views in Cost Analysis:
- **Cost by Resource:** Identifies the top 5 most expensive single resources (e.g., a neglected SQL database or unattached Premium SSD disk).
- **Cost by Service:** Breaks down spend between Compute, Storage, Networking, and Database services.
- **Cost by Tag:** Allows the finance department to view exactly how much the `Marketing` team spent vs. the `DevOps` team this month.
- **Forecasting:** Azure uses machine learning to project your estimated month-end bill based on your current burn rate.

---

## 4. Hands-on Lab: Setting Up Budgets & Automated Cost Alerts

Preventing surprise bills is mandatory for every cloud engineer. Follow these steps to configure a hard budget alert:

### Step 1: Navigate to Cost Management
1. In the Azure Portal search bar, type **Cost Management + Billing**.
2. Select your subscription (e.g., *Azure subscription 1*).
3. In the left navigation menu under **Cost management**, click **Budgets**.

---

### Step 2: Create a Monthly Budget
1. Click **+ Add**.
2. **Budget Details:**
   - **Name:** `Monthly-Dev-Budget`.
   - **Reset period:** `Monthly` (Resets on the 1st of every month).
   - **Creation date:** Today's date.
   - **Expiration date:** 2 years from now.
   - **Budget amount:** Enter your limit, e.g., `$100.00` (or `₹5,000`).
3. Click **Next**.

---

### Step 3: Configure Alert Thresholds
Azure allows multiple alert tiers based on percentage of budget consumed:

```
+-------------------------------------------------------------------+
| BUDGET THRESHOLD ALERT MATRIX                                     |
+-------------------------------------------------------------------+
|  Threshold %  | Amount | Action Triggered                         |
|  -------------+--------+----------------------------------------- |
|  50%          | $50.00 | Informational email sent to developer    |
|  80%          | $80.00 | Warning email sent to engineering lead   |
|  100%         | $100.0 | Critical email + Trigger Action Group    |
|               |        | (Automated script stops non-prod VMs!)   |
+-------------------------------------------------------------------+
```

1. **Alert 1:**
   - `% of budget`: `50`
   - Action group: (Optional)
   - Email recipients: Enter your personal email address.
2. **Alert 2:**
   - `% of budget`: `80`
   - Email recipients: Enter your email address.
3. Click **Create**.
4. Azure will continuously evaluate your spending hourly and notify you immediately if costs spike!

---

## 5. Azure Advisor: Automated Cost Optimization

**Azure Advisor** is a built-in AI assistant that scans your environment against best practices:
- **Cost Recommendations:** Flags virtual machines with average CPU utilization under 5% and suggests resizing them to burstable B-series or deallocating them.
- **Unattached Disks:** Identifies storage disks that were left behind after a VM was deleted.
- **Reserved Instances:** Recommends purchasing 1-year or 3-year Azure Reservations to save up to 72% on steady-state production workloads.

---

## 6. Summary Checklist
- [x] Azure Tags provide metadata essential for departmental chargeback and resource tracking.
- [x] Cost Analysis enables slicing cloud expenses by service, resource group, and custom tags.
- [x] Budgets alert engineers when monthly spending crosses defined percentage thresholds.
- [x] Azure Advisor continuously recommends resizing idle VMs and purging orphaned storage disks.
