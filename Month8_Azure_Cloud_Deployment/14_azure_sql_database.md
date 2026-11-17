# 14 - Azure SQL Database (RDBMS)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `06:22:08` to `06:50:47`

---

## 1. What is an RDBMS?

A **Relational Database Management System (RDBMS)** stores structured data into organized tables composed of rows and columns. 

Tables relate to one another through **Primary Keys** and **Foreign Keys** (e.g., a `Customers` table linked to an `Orders` table). Relational databases enforce strict **ACID transactions** (Atomicity, Consistency, Isolation, Durability) so financial transfers and inventory counts never get corrupted.

---

## 2. The Three Ways to Run SQL on Microsoft Azure

```
Control & Maintenance                                                    Automated & Managed
--------------------------------------------------------------------------------------------
[ SQL on Azure VMs (IaaS) ]  --->  [ SQL Managed Instance (PaaS) ]  --->  [ Azure SQL Database (PaaS) ]
- You manage OS & SQL Server        - Managed instance with 100%          - Pure Database as a Service
- You handle backups & patching       on-prem feature compatibility        - Zero OS or instance management
```

### 1. SQL Server on Azure VMs (IaaS)
- You deploy a Windows/Linux VM and install Microsoft SQL Server on it.
- You have 100% control, but you are responsible for OS updates, database version patches, manual backups, and high availability clustering.

### 2. Azure SQL Managed Instance (PaaS)
- A fully managed SQL instance designed for enterprise companies migrating legacy on-premises SQL Server databases to the cloud with nearly 100% feature compatibility (supports SQL Agent, Cross-database queries, CLR).

### 3. Azure SQL Database (Database-as-a-Service)
- **The modern, cloud-native choice:** Microsoft manages the operating system, SQL engine, backups, replication, and high availability behind the scenes.
- You don't create or manage a database server; you simply create a database and start running SQL queries!
- Backed by an industry-leading **99.995% availability SLA**.

---

## 3. Purchasing Models: DTU vs. vCore & Serverless

When creating an Azure SQL Database, Azure offers two pricing models:

### 1. DTU Model (Database Transaction Unit)
- A simplified, pre-packaged bundle of compute (CPU), memory (RAM), and I/O.
- Tiers: **Basic** (super cheap, $5/mo, 2 GB storage, best for students), **Standard**, and **Premium**.
- Simplest for beginners who don't want to calculate exact CPU cores.

### 2. vCore Model (Virtual Core)
- Lets you independently choose your exact number of CPU cores and storage in Gigabytes.
- Includes the revolutionary **Serverless Tier**:
  - Automatically scales compute up and down based on real-time query activity.
  - **Auto-pauses the database when no queries are running**—billing for compute drops to **$0** while paused!
  - Wakes up automatically when a new query arrives.

---

## 4. Key Security Features in Azure SQL

1. **Firewall Rules:**
   - By default, Azure SQL blocks all internet traffic.
   - You must add your client laptop IP address to the firewall rules to connect.
   - You can toggle *"Allow Azure services and resources to access this server"* to let your Azure VMs, App Services, and Container Apps connect.
2. **Transparent Data Encryption (TDE):**
   - Automatically encrypts all data stored on disk and in database backups at rest using AES-256 bit keys.
3. **Microsoft Entra ID (Azure AD) Authentication:**
   - Connect using cloud corporate accounts instead of managing database passwords.
4. **Automated Backups & Point-in-Time Restore:**
   - Azure takes full backups weekly, differential backups every 12 hours, and transaction log backups every 10 minutes.
   - You can restore your database to any exact minute in the past 7 to 35 days!

---

## 5. Hands-on Walkthrough: Deploying & Querying Azure SQL

In the video, M Prashant guides us through creating a database and querying it directly from the Azure Portal:

### Step 1: Create Azure SQL Database
1. In the Azure Portal search bar, type **SQL databases** -> Click **Create**.
2. **Project Details:**
   - Resource Group: `rg-database-demo`.
   - Database name: `StudentDB`.
3. **Server:** Click *Create new*:
   - Server name: `student-sql-server-$RANDOM` (globally unique).
   - Region: `East US`.
   - Authentication: Select **Use SQL authentication**.
   - Admin login: `azureadmin`.
   - Password: Enter a strong password. Click **OK**.
4. **Compute + Storage:**
   - Click *Configure database*.
   - Select **Basic** (5 DTUs, 2 GB storage, ~$5/month) to avoid high charges.
5. Click **Review + Create**, then click **Create**.

### Step 2: Configure Server Firewall
1. Once deployed, click **Go to resource**.
2. In the top toolbar, click **Set server firewall**.
3. Click **+ Add your client IPv4 address** (adds your current home/office IP).
4. Set *"Allow Azure services and resources to access this server"* to **Yes**.
5. Click **Save**.

### Step 3: Querying the Database in Azure Portal (Query Editor)
1. On the left menu of `StudentDB`, click **Query editor (preview)**.
2. Enter your SQL admin login (`azureadmin`) and password -> Click **OK**.
3. In the query window, run standard SQL statements:

```sql
-- 1. Create a table
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Course VARCHAR(50)
);

-- 2. Insert sample records
INSERT INTO Students VALUES (101, 'Rahul', 'Sharma', 'Cloud Computing');
INSERT INTO Students VALUES (102, 'Priya', 'Verma', 'DevOps & Azure');

-- 3. Query the data
SELECT * FROM Students;
```

Click **Run**. The query results immediately appear in the bottom pane, demonstrating a fully functional, production-ready relational database on Azure!

---

## 6. Summary Checklist
- [x] Azure SQL Database is a fully managed PaaS relational database with 99.995% uptime SLA.
- [x] DTU model is simple and cheap for beginners; vCore Serverless autoscales and pauses when idle.
- [x] Firewall rules restrict access strictly to whitelisted client IPs and Azure internal services.
- [x] Built-in automatic backups, Transparent Data Encryption (TDE), and Point-in-Time recovery.
- [x] Query directly from the browser using Azure Portal Query Editor without local software.
