# 14 - Relational Databases: Azure SQL Database (PaaS RDBMS)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `06:22:08` to `06:50:47`

---

## 1. Relational Databases (RDBMS) in the Cloud

A **Relational Database Management System (RDBMS)** organizes data into structured tables consisting of rows and columns with enforced relationships (Foreign Keys) and strict **ACID transactions** (Atomicity, Consistency, Isolation, Durability).

When running SQL databases on Azure, you have three distinct hosting models:

```
+--------------------------------------------------------------------------+
|                        AZURE SQL DEPLOYMENT OPTIONS                      |
+--------------------------------------------------------------------------+
|                                                                          |
|  1. SQL SERVER ON AZURE VM (IaaS)                                        |
|     - You manage: OS patching, SQL licensing, backups, storage volumes.  |
|     - Best for: Full OS control or legacy OS dependencies.               |
|                                                                          |
|  2. AZURE SQL MANAGED INSTANCE (PaaS)                                    |
|     - 100% compatibility with on-premises SQL Server instances.          |
|     - Native VNet integration, SQL Agent jobs, cross-database queries.   |
|                                                                          |
|  3. AZURE SQL DATABASE (Fully Managed PaaS)                              |
|     - Zero OS or server management.                                      |
|     - Built-in High Availability (99.99% SLA), automated backups.        |
|     - Serverless auto-pause (stops billing when idle!).                  |
|                                                                          |
+--------------------------------------------------------------------------+
```

---

## 2. Azure SQL Database: Key Architectures

### Purchasing Models:
1. **vCore-based Model (Recommended):** Provides independent, granular control over compute (number of virtual CPU cores) and storage size.
   - **General Purpose:** Standard production workloads with budget-friendly SSD storage.
   - **Business Critical:** Low-latency transactional workloads with in-memory storage and multi-replica availability.
   - **Hyperscale:** Highly scalable storage up to 100 TB with instant backups and rapid restore.
2. **DTU-based Model (Legacy):** Bundles CPU, memory, and read/write I/O into a single "Database Transaction Unit" metric.

### Serverless Compute Tier (Budget Saver):
The **Serverless tier** automatically scales compute resources based on workload demand and **automatically pauses the database during inactive periods**:
- While paused, you pay strictly for storage, saving up to 75% on compute bills!
- When a new SQL query arrives, the database wakes up automatically within seconds.

---

## 3. Hands-on Lab: Deploying Azure SQL & Running Queries

### Step 1: Create Azure SQL Database in Portal
1. Open the [Azure Portal](https://portal.azure.com).
2. Search for **SQL databases** and click **+ Create**.
3. **Basics Tab:**
   - **Subscription:** Select your subscription.
   - **Resource group:** `rg-database-demo`.
   - **Database name:** `sqldb-enterprise-demo`.
   - **Server:** Click *Create new*:
     - Server name: `sqlserver-demo-12345` *(Must be globally unique)*.
     - Location: `East US`.
     - Authentication method: **Use SQL authentication**.
     - Server admin login: `sqladmin`.
     - Password: `ComplexPassword!2026`.
     - Click **OK**.
   - **Compute + storage:** Click *Configure database*:
     - Service tier: **General Purpose (Serverless)**.
     - Min vCores: `0.5`, Max vCores: `1`.
     - Check `[x] Enable auto-pause` -> Delay: `1 hour`.
     - Click **Apply**.
4. Click **Review + create** -> **Create**. (Takes ~2 minutes).

---

### Step 2: Configure Server Firewall Rules
By default, Azure SQL blocks all incoming internet traffic to safeguard your data:

1. Open your new SQL Database and click **Set server firewall** at the top.
2. **Public network access:** Ensure it is set to **Selected networks**.
3. **Allow Azure services:** Check `[x] Allow Azure services and resources to access this server` (This enables your Azure VMs and App Services to connect).
4. **Add Client IP:** Click **+ Add your client IPv4 address** (Adds your laptop's current public IP).
5. Click **Save**.

---

### Step 3: Run Queries via the Portal "Query Editor"
You can interact with your database directly inside the Azure Portal without installing any software:

1. In the left navigation menu of your SQL Database, click **Query editor (preview)**.
2. Log in using your credentials:
   - **Login:** `sqladmin`
   - **Password:** `ComplexPassword!2026`
   - Click **OK**.
3. In the query window, execute standard T-SQL commands:

```sql
-- 1. Create a table
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY IDENTITY(1,1),
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Email VARCHAR(100) UNIQUE,
    CreatedDate DATETIME DEFAULT GETDATE()
);

-- 2. Insert sample records
INSERT INTO Customers (FirstName, LastName, Email)
VALUES 
    ('Abhishek', 'Veeramalla', 'abhishek@example.com'),
    ('Prashant', 'Kumar', 'prashant@example.com'),
    ('John', 'Doe', 'john.doe@example.com');

-- 3. Query the data
SELECT * FROM Customers;
```

4. Click **Run**. The results grid instantly displays your records!

---

## 4. Connecting from External Tools (SSMS & Code)

### Connection String:
Navigate to the **Connection strings** tab in the portal to retrieve pre-formatted strings for ADO.NET, JDBC, ODBC, or PHP:
```
Server=tcp:sqlserver-demo-12345.database.windows.net,1433;Initial Catalog=sqldb-enterprise-demo;Persist Security Info=False;User ID=sqladmin;Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

---

## 5. Summary Checklist
- [x] Azure SQL Database is a fully managed PaaS relational engine with built-in 99.99% availability.
- [x] Serverless compute tier automatically scales down and pauses when idle to eliminate compute costs.
- [x] Server firewall rules restrict access by client IP and manage Azure-internal network access.
- [x] Built-in Query Editor allows rapid schema design, querying, and verification directly in the browser.
