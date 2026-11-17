# 15 - Data Engineering: Azure Data Factory (ADF) & ETL Pipelines

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `06:50:47` to `07:19:01`

---

## 1. What is ETL and Data Integration?

Enterprises accumulate massive volumes of data across isolated systems: legacy on-premises SQL databases, external SaaS tools (Salesforce, SAP), IoT devices, and cloud object stores (AWS S3, Azure Blob).

To derive business intelligence, data must be gathered into a centralized repository (like Azure Synapse Analytics or a Data Lake):

```
+--------------------------------------------------------------------------+
|                         ETL VS. ELT DATA FLOW                            |
+--------------------------------------------------------------------------+
|                                                                          |
|  EXTRACT (E):      Read raw data from 90+ diverse sources.               |
|                    (e.g., MySQL, Oracle, On-prem files, REST APIs)       |
|                                                                          |
|  TRANSFORM (T):    Cleanse, filter, aggregate, anonymize, and reformat.  |
|                                                                          |
|  LOAD (L):         Write structured results into target data warehouses. |
|                    (e.g., Azure SQL Database, Synapse, Snowflake)        |
|                                                                          |
+--------------------------------------------------------------------------+
```

---

## 2. What is Azure Data Factory (ADF)?

**Azure Data Factory (ADF)** is Microsoft's fully managed, serverless cloud data integration and workflow orchestration service. It allows you to build complex automated data pipelines visually without writing boilerplate integration code.

### Core Building Blocks of Azure Data Factory:

```
+-------------------------------------------------------------------------+
|                  AZURE DATA FACTORY CORE ARCHITECTURE                   |
+-------------------------------------------------------------------------+
|                                                                         |
|  [ TRIGGER ] (Scheduled at 2 AM / Blob Created Event)                   |
|       |                                                                 |
|       v                                                                 |
|  [ PIPELINE ]                                                           |
|       |                                                                 |
|       +---> [ Activity: Lookup metadata ]                               |
|       |                                                                 |
|       +---> [ Activity: Copy Data Activity ]                            |
|                  |                     |                                |
|                  v (Source)            v (Sink)                         |
|             [ Dataset A ]         [ Dataset B ]                         |
|                  |                     |                                |
|                  v                     v                                |
|         [ Linked Service 1 ]  [ Linked Service 2 ]                      |
|         (Azure Blob Storage)  (Azure SQL Database)                      |
|                                                                         |
|  COMPUTE ENGINE:                                                        |
|  [ Integration Runtime (IR) ] (Azure Managed IR or Self-Hosted IR)      |
|                                                                         |
+-------------------------------------------------------------------------+
```

### Definitions:
1. **Linked Services:** The connection manager (defines connection string, authentication keys, and server address to external data stores).
2. **Datasets:** Named pointers that specify the exact file or table structure within a Linked Service (e.g., `customers.csv` inside container `raw-data`).
3. **Activities:** Discrete execution steps in a pipeline (e.g., *Copy Data*, *Execute Stored Procedure*, *Notebook execution*, *If Condition*).
4. **Pipelines:** Logical groupings of activities designed to perform a complete task.
5. **Integration Runtime (IR):** The compute infrastructure executing the data movement.
   - **Azure IR:** Fully managed, serverless, scales automatically in Azure.
   - **Self-Hosted IR:** Installed on an on-premises physical server or VM to securely pull data from corporate networks behind enterprise firewalls!

---

## 3. Hands-on Lab: Building an Automated Copy Pipeline (Blob to SQL)

In this lab from the video, M Prashant demonstrates extracting a customer CSV file from Blob Storage and loading it into an Azure SQL Database table:

### Step 1: Create the Azure Data Factory Resource
1. Open the [Azure Portal](https://portal.azure.com).
2. Search for **Data factories** and click **+ Create**.
3. **Basics Tab:**
   - **Subscription:** Select your subscription.
   - **Resource group:** `rg-data-demo`.
   - **Name:** `adf-etl-masterclass-demo`.
   - **Region:** `East US`.
   - **Version:** `V2`.
4. Click **Review + create** -> **Create**.

---

### Step 2: Launch ADF Studio
1. Open your newly created Data Factory.
2. Click **Launch studio** in the center tile.
3. ADF Studio opens in a specialized full-screen engineering workspace (`https://adf.azure.com`).

---

### Step 3: Configure Linked Services (Connections)
1. In ADF Studio, click the **Manage** toolbox icon (wrench) on the far left.
2. Under **Connections**, click **Linked services** -> **+ New**:
   - **Connection 1 (Source):** Select **Azure Blob Storage** -> Name: `ls_BlobStorage` -> Select your Storage Account -> Test connection -> Click **Create**.
   - **Connection 2 (Sink/Target):** Select **Azure SQL Database** -> Name: `ls_AzureSqlDb` -> Select your SQL Server and Database -> Enter SQL admin credentials -> Test connection -> Click **Create**.

---

### Step 4: Create the Data Pipeline
1. Click the **Author** pencil icon on the left menu.
2. Click **+** -> **Pipeline** -> **Pipeline**.
3. In the Activities toolbox, expand **Move & transform** and drag the **Copy data** activity onto the designer canvas.

```
+-------------------------------------------------------------------+
| PIPELINE CANVAS: Copy Data Activity                               |
|                                                                   |
| [ SOURCE TAB ]                 ===>  [ SINK TAB ]                 |
| - Dataset: ds_CustomerCSV            - Dataset: ds_SqlCustomers   |
| - Path: raw-data/customers.csv       - Table: dbo.Customers       |
|                                                                   |
|                      [ MAPPING TAB ]                              |
|           CSV Column "cust_id"    ->  SQL Column "CustomerID"     |
|           CSV Column "first_name" ->  SQL Column "FirstName"      |
|           CSV Column "email"      ->  SQL Column "Email"          |
+-------------------------------------------------------------------+
```

---

### Step 5: Debug & Trigger Pipeline
1. At the top of the canvas, click **Debug** to execute a dry run.
2. Check the **Output** tab at the bottom:
   - Status updates from *Queued* -> *In Progress* -> **Succeeded**.
   - View detailed metrics: Rows read (1,000), Rows written (1,000), Data throughput (2.4 MB/s).
3. Open your Azure SQL Query Editor and run:
   ```sql
   SELECT COUNT(*) FROM Customers;
   ```
   All 1,000 customer records have been transformed and loaded into your relational database!

---

## 4. Summary Checklist
- [x] Azure Data Factory orchestrates serverless ETL/ELT pipelines across hybrid cloud environments.
- [x] Pipelines contain Activities (such as Copy Data, Data Flows, and Databricks jobs).
- [x] Linked Services store connection credentials; Datasets define target tables and files.
- [x] Self-Hosted Integration Runtimes bridge on-premises enterprise datacenters to Azure securely.
- [x] Pipelines can be automated using schedule, tumbling window, or storage event triggers.
