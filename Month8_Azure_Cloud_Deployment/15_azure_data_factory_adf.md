# 15 - Azure Data Factory (ADF & ETL Pipelines)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `06:50:47` to `07:19:01`

---

## 1. What is ETL and Data Integration?

In modern companies, data lives in dozens of scattered places:
- Daily sales receipts in an on-premises Oracle database.
- Web clickstream logs stored as `.csv` files in Azure Blob Storage.
- Customer support tickets inside Salesforce.
- Marketing stats inside Google Analytics.

If a company's CEO wants a single dashboard showing total daily profit and customer churn, data engineers cannot copy-paste millions of records by hand.

They need an automated **ETL (Extract, Transform, Load)** pipeline:
1. **Extract:** Read data from multiple disparate sources.
2. **Transform:** Clean messy data, filter out invalid rows, mask credit card numbers, and join tables.
3. **Load:** Write the clean, structured data into a central Data Warehouse (like Azure Synapse Analytics or Azure SQL) for Power BI reports.

---

## 2. What is Azure Data Factory (ADF)?

**Azure Data Factory (ADF)** is Microsoft's cloud-based, serverless data integration and workflow orchestration service.

Think of Azure Data Factory as the **central railway station or plumbing system for big data**:
- It doesn't store data itself.
- Instead, it orchestrates moving and transforming massive amounts of data across 90+ pre-built connectors (AWS S3, Google Cloud, Oracle, MySQL, Azure Blob, Salesforce, SAP, and Snowflake).
- It is completely serverless: you pay only for the minutes your data movement pipelines are actively running.

---

## 3. The 6 Core Building Blocks of Azure Data Factory

```
+-------------------------------------------------------------------------------+
|                             AZURE DATA FACTORY                                |
|                                                                               |
|  [ TRIGGER ] (e.g., Every Midnight or On File Upload)                         |
|      |                                                                        |
|      v                                                                        |
|  [ PIPELINE ]                                                                 |
|      |                                                                        |
|      v                                                                        |
|  [ ACTIVITY: Copy Data ] <------------ [ INTEGRATION RUNTIME (Compute) ]      |
|      |                                                                        |
|      +--------> Reads from: [ SOURCE DATASET ] ---> [ Linked Service to Blob ]|
|      |                                                                        |
|      +--------> Writes to:  [ SINK DATASET ]   ---> [ Linked Service to SQL ] |
+-------------------------------------------------------------------------------+
```

### 1. Linked Service (The Connection String)
Defines the connection information (server address, credentials, authentication) needed to connect to an external source or destination. 
- *Example:* A Linked Service pointing to your Azure Blob Storage account, or a Linked Service connecting to your Azure SQL Database.

### 2. Dataset (The Data Structure)
A named view of the data pointing directly to a file, folder, or table specified by a Linked Service.
- *Example:* A dataset named `SalesCSV_Dataset` pointing to `/data/sales_2026.csv` inside Azure Blob Storage.

### 3. Activity (The Action Step)
A specific processing step executed inside a pipeline. Common activities:
- **Copy Activity:** Moves data from Source to Sink (Destination).
- **Data Flow Activity:** Visually transforms and cleans data without writing code.
- **Web Activity:** Calls an external REST API or webhook.
- **Databricks / Stored Procedure Activity:** Triggers external code execution.

### 4. Pipeline (The Logical Workflow)
A collection of activities grouped together to accomplish a business task (e.g., *DailyCustomerDataIngestion*).

### 5. Integration Runtime (IR - The Compute Engine)
The actual computing infrastructure that executes the data movement.
- **Azure IR:** Fully managed by Microsoft (moves cloud-to-cloud data).
- **Self-Hosted IR:** Installed on an on-premises server to pull data through private firewalls into Azure.

### 6. Trigger (The Starter)
Determines *when* a pipeline executes:
- **Schedule Trigger:** Runs on a periodic timer (e.g., every 6 hours).
- **Tumbling Window:** Evaluates historical time-slice data.
- **Event-Based Trigger:** Fires automatically when a file is uploaded to Blob storage.

---

## 4. Hands-on Walkthrough: Building a Blob-to-SQL Copy Pipeline

In the video, M Prashant demonstrates a classic real-world ETL pipeline copying data from a CSV file in Blob Storage directly into an Azure SQL database table:

### Step 1: Create an Azure Data Factory Instance
1. In the Azure Portal search bar, type **Data factories** -> Click **Create**.
2. Resource Group: `rg-data-demo`.
3. Name: `adf-student-demo-$RANDOM`.
4. Region: `East US`.
5. Version: **V2**.
6. Click **Review + Create** -> **Create**.
7. Once deployed, click **Launch studio** to open the Azure Data Factory visual designer interface.

### Step 2: Create Linked Services
Inside Azure Data Factory Studio:
1. Click the **Manage** tab (wrench icon on the far left).
2. Click **Linked services** -> **+ New**:
   - Create Linked Service 1: Select **Azure Blob Storage**, authenticate to your storage account, click *Test connection* -> *Create*.
   - Create Linked Service 2: Select **Azure SQL Database**, enter your server and `StudentDB` credentials, click *Test connection* -> *Create*.

### Step 3: Build the Copy Pipeline
1. Click the **Author** tab (pencil icon on the far left).
2. Click **+** -> **Pipeline** -> Name it `CopyBlobToSqlPipeline`.
3. In the Activities toolbox, expand **Move & transform** and drag the **Copy data** activity onto the canvas.
4. **Source Tab:**
   - Source dataset: Click *+ New* -> Choose *Azure Blob Storage* -> *DelimitedText (CSV)*.
   - Point to your uploaded `students.csv` file. Check *First row as header*.
5. **Sink Tab (Destination):**
   - Sink dataset: Click *+ New* -> Choose *Azure SQL Database*.
   - Table: Select `dbo.Students`.
6. **Mapping Tab:**
   - Click **Import schemas**. ADF automatically matches the CSV columns (`StudentID`, `FirstName`, `LastName`, `Course`) to your SQL table columns!

### Step 4: Debug & Run
1. In the top toolbar, click **Debug**.
2. ADF provisions an Azure Integration Runtime, reads the CSV rows from Blob Storage, and writes them into Azure SQL.
3. Once the status shows **Succeeded**, switch to your Azure SQL Query Editor and execute:
   ```sql
   SELECT * FROM Students;
   ```
   All the rows from the CSV file are now safely stored inside your relational database!

---

## 5. Summary Checklist
- [x] ETL stands for Extract, Transform, and Load—essential for modern data pipelines.
- [x] Azure Data Factory (ADF) orchestrates data movement across 90+ connectors.
- [x] Know the 6 core components: Linked Services, Datasets, Activities, Pipelines, Integration Runtimes, and Triggers.
- [x] Use Azure IR for cloud-to-cloud transfers; use Self-Hosted IR for on-premise migrations.
