# 16 - Azure Tags, Billing & Cost Management

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `07:19:01` to `07:27:57`

---

## Part 1: Azure Tags (Resource Organization & Tracking)

### 1. What are Azure Tags?
When a company runs hundreds of servers, databases, and storage accounts across multiple continents, the Azure Portal quickly becomes a cluttered mess. If the monthly bill arrives at $25,000, how does the finance department know how much was spent by the Mobile App team versus the Machine Learning team?

**Azure Tags** are metadata labels—stored as **Name-Value (Key-Value) pairs**—that you attach directly to Azure resources, resource groups, and subscriptions.

```
+-------------------------------------------------------------------------------+
|                            RESOURCE: my-production-vm                         |
|                                                                               |
|  TAG NAME (KEY)           TAG VALUE                                           |
|  -------------------------------------------------------------                |
|  Environment              "Production"                                        |
|  Project                  "E-Commerce-Store"                                  |
|  Owner                    "rahul.sharma@company.com"                          |
|  CostCenter               "Finance-Dept-402"                                  |
|  AutoShutdownSchedule     "Never"                                             |
+-------------------------------------------------------------------------------+
```

---

### 2. Why DevOps Engineers Use Tags
1. **Cost Allocation & Billing Reports:** Filter your monthly cloud bill by `CostCenter` or `Project` so finance can charge the correct department.
2. **Operations & Automation:** A nightly script can search for all VMs with `Environment: Dev` and automatically shut them down at 8:00 PM to save money.
3. **Governance & Compliance:** Prevent orphaned resources by mandating an `Owner` tag on every deployed resource.
4. **Limits:** A single resource can have up to **50 tags**. Tags are not case-sensitive for operations, but preserve casing for reporting.

> **Important Note:** Unlike RBAC permissions, **Tags are NOT automatically inherited** from a Resource Group down to child resources! You can use an **Azure Policy** to automatically copy tags downward.

---

## Part 2: Azure Cost Management & Billing

### 1. The #1 Fear of Cloud Beginners: Unexpected Bills!
Every beginner worries about accidentally leaving an expensive server running and receiving a surprise $500 bill at the end of the month.

Microsoft Azure provides **Azure Cost Management and Billing** to monitor, track, analyze, and strictly control cloud spending.

---

### 2. Cost Analysis: Visualizing Where Your Money Goes
Inside the Azure Portal, search for **Cost Management + Billing** -> Click **Cost analysis**:
- View your **Accumulated Cost** chart showing actual daily spending.
- View **Forecasted Cost** (Azure AI predicts what your bill will be at the end of the month based on your current usage rate).
- **Filter and Group:** Group by *Resource Type*, *Service Name*, *Location*, or *Tag* to see exactly which service is burning your budget.

---

### 3. Setting Up Azure Budgets & Automated Alerts

Every student and engineer should set up a **Budget Alert** on Day 1:

```
[ Your Monthly Budget: $50 ]
           |
           +---> 50% Spent ($25) ---> Sends Warning Email ("You are at 50%")
           |
           +---> 80% Spent ($40) ---> Sends Urgent Email ("Slow down!")
           |
           +---> 100% Spent ($50) ---> Sends Alert / Triggers Action Group
```

### Step-by-Step Setup:
1. In the Azure Portal, open **Cost Management + Billing** -> Select your subscription.
2. In the left menu, click **Budgets** -> Click **+ Add**.
3. **Budget Details:**
   - Name: `StudentMonthlyBudget`.
   - Reset period: **Monthly**.
   - Amount: Enter your limit (e.g., `$20` or `$50`).
4. **Alert Conditions:**
   - Alert 1: `50%` of budget -> Enter your email address.
   - Alert 2: `80%` of budget -> Enter your email address.
   - Alert 3: `100%` of budget -> Enter your email address.
5. Click **Create**.

Now, if a test script accidentally spins up multiple resources, Azure will email you immediately before the bill gets out of hand!

---

## 4. Golden Rules for Students to Save Cloud Money

1. **Always Stop / Deallocate VMs:**
   - In the Azure Portal, clicking **Stop** on a VM puts it into the **`Stopped (Deallocated)`** state.
   - In this state, **Microsoft stops charging you for CPU and RAM compute!** (You only pay a few pennies for the disk storage).
2. **Delete Resource Groups After Labs:**
   - When you finish practicing a tutorial, delete the entire Resource Group (`rg-aks-demo`). This wipes out all associated VMs, disks, public IPs, and load balancers in one click.
3. **Use Azure Advisor:**
   - Search for **Azure Advisor** in the portal. Click **Cost**.
   - Azure scans your account and flags:
     - Unused public IP addresses costing money.
     - Orphaned virtual machine disks left behind after VM deletion.
     - Underutilized VMs that should be downsized to `B1s`.

---

## 5. Summary Checklist
- [x] Azure Tags attach key-value metadata to track ownership, projects, and cost centers.
- [x] Tags do not inherit automatically down to child resources by default.
- [x] Use Cost Analysis to visualize daily spending and forecasted monthly totals.
- [x] Always configure Budgets and percentage threshold email alerts on every subscription.
- [x] Deallocate VMs and delete lab resource groups when not in use.
