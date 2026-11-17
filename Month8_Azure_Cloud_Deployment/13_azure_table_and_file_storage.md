# 13 - Specialized Storage: Azure Table Storage (NoSQL) & Azure Files (SMB/NFS)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `05:57:26` to `06:22:08`

---

## 1. What is Azure Table Storage (NoSQL)?

**Azure Table Storage** is a low-cost, highly available NoSQL key-value datastore. Unlike traditional relational databases (like MySQL or SQL Server), Table Storage is **schemaless**: each record (called an **entity**) can hold completely different properties (columns) from every other record.

```
+-------------------------------------------------------------------------+
|                  AZURE TABLE STORAGE DATA MODEL                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  PartitionKey | RowKey    | Timestamp           | Custom Properties     |
|  -------------+-----------+---------------------+---------------------- |
|  "CustomerUS" | "001"     | 2026-11-17T10:00:00 | Name="Alice", Age=28  |
|  "CustomerUS" | "002"     | 2026-11-17T10:01:00 | Name="Bob", City="NY" |
|  "CustomerEU" | "101"     | 2026-11-17T10:02:00 | Name="Eve", VIP=True  |
|                                                                         |
|  * PartitionKey + RowKey together form the unique PRIMARY KEY!          |
+-------------------------------------------------------------------------+
```

### The Composite Primary Key:
Every entity in an Azure Table requires three mandatory system properties:
1. **PartitionKey (String):** Groups related entities together onto the same physical storage partition. Queries scoped to a single `PartitionKey` execute with sub-second speed.
2. **RowKey (String):** The unique identifier for an entity within a given partition.
3. **Timestamp (DateTime):** Automatically maintained by Azure to support optimistic concurrency control.

### When to Use Table Storage vs. Azure Cosmos DB:
- **Choose Table Storage:** When storing terabytes of structured logs, telemetry, IoT sensor snapshots, or simple user profiles where cost must remain extremely low ($0.045 per GB/month).
- **Choose Azure Cosmos DB:** When you require global multi-region active-active replication, guaranteed single-digit millisecond latency SLAs, or complex SQL/MongoDB/Gremlin APIs.

---

## 2. Hands-on Lab: Working with Azure Table Storage

### Step 1: Create a Table
1. In the Azure Portal, open your Storage Account (`mystoragedemo12345`).
2. Under **Data storage**, click **Tables**.
3. Click **+ Table** -> Name: `DeviceTelemetry` -> Click **OK**.

---

### Step 2: Insert Entities (Schemaless Demonstration)
1. Inside the `DeviceTelemetry` table, click **+ Add entity**:
   - **PartitionKey:** `Building-A`
   - **RowKey:** `Sensor-101`
   - Click **Add property**: Name: `Temperature`, Type: `Double`, Value: `72.5`
   - Click **Add property**: Name: `Status`, Type: `String`, Value: `Normal`
   - Click **Insert**.
2. Click **+ Add entity** again:
   - **PartitionKey:** `Building-A`
   - **RowKey:** `Camera-202`
   - Click **Add property**: Name: `Resolution`, Type: `String`, Value: `4K`
   - Click **Add property**: Name: `BatteryPercent`, Type: `Int32`, Value: `88`
   - Click **Insert**.

Notice how Sensor 101 and Camera 202 store completely different fields in the same table without needing schema migrations or altering tables!

---

## 3. What is Azure Files (Cloud File Shares)?

**Azure Files** provides fully managed cloud file shares that can be accessed using standard industry protocols:
- **SMB (Server Message Block - Port 445):** Mountable on Windows, macOS, and Linux laptops or servers.
- **NFS (Network File System - Port 2049):** High-performance mounting on Linux enterprise servers.

```
+-------------------------------------------------------------------------+
|                       AZURE FILES CLOUD SHARING                         |
+-------------------------------------------------------------------------+
|                                                                         |
|                          +--------------------+                         |
|                          |  AZURE FILE SHARE  |                         |
|                          |   (Storage Acct)   |                         |
|                          +---------+----------+                         |
|                                    |                                    |
|               +--------------------+--------------------+               |
|               | SMB Protocol (Port 445)                 |               |
|               v                                         v               |
|       +----------------+                       +----------------+       |
|       | Windows Server |                       | Linux VM       |       |
|       | Mounted as Z:\ |                       | Mounted at     |       |
|       |                |                       | /mnt/clouddata |       |
|       +----------------+                       +----------------+       |
|                                                                         |
+-------------------------------------------------------------------------+
```

### Why Azure Files?
- **"Lift and Shift" Migration:** Move legacy on-premises applications to the cloud without rewriting their code. If an app reads files from `\\fileserver\shared`, you simply redirect it to Azure Files!
- **Shared Access:** Dozens of virtual machines can read and write to the exact same file share concurrently.
- **Persistent Storage for Containers:** Easily mounted into Azure Kubernetes Service (AKS) pods as persistent volumes.

---

## 4. Hands-on Lab: Creating and Mounting an Azure File Share

### Step 1: Create the File Share
1. In your Storage Account, under **Data storage**, click **File shares**.
2. Click **+ File share**:
   - **Name:** `corporate-share`.
   - **Tier:** **Transaction optimized** (for standard practice) or **Hot**.
   - Click **Review + create** -> **Create**.

---

### Step 2: Retrieve the Auto-Generated Mount Script
1. Click on `corporate-share`.
2. Click the **Connect** button at the top:
   - Azure generates a ready-to-run connection script for **Windows**, **Linux**, or **macOS**!

#### For Windows (PowerShell):
```powershell
$connectTestResult = Test-NetConnection -ComputerName "mystoragedemo12345.file.core.windows.net" -Port 445
if ($connectTestResult.TcpTestSucceeded) {
    New-PSDrive -Name "Z" -PSProvider "FileSystem" -Root "\\mystoragedemo12345.file.core.windows.net\corporate-share" -Persist -Credential (New-Object System.Management.Automation.PSCredential ("mystoragedemo12345", (ConvertTo-SecureString "YOUR_STORAGE_KEY" -AsPlainText -Force)))
} else {
    Write-Error -Message "Unable to reach the Azure storage account via port 445."
}
```

#### For Linux (Ubuntu / Debian):
```bash
# 1. Install CIFS utilities
sudo apt-get update
sudo apt-get install -y cifs-utils

# 2. Create the local mount directory
sudo mkdir -p /mnt/corporate-share

# 3. Mount the Azure File Share using SMB credentials
sudo mount -t cifs //mystoragedemo12345.file.core.windows.net/corporate-share /mnt/corporate-share -o vers=3.0,username=mystoragedemo12345,password=YOUR_STORAGE_KEY,dir_mode=0777,file_mode=0777,serverino
```

3. Any file written to `/mnt/corporate-share` on the Linux VM is immediately visible on the Windows `Z:\` drive in real time!

---

## 5. Troubleshooting Azure Files & Table Storage

| Problem | Cause | Resolution |
| :--- | :--- | :--- |
| Windows mounting fails with `Port 445 blocked`. | Residential ISPs frequently block outbound port 445 (SMB) to stop malware. | Connect via an Azure VPN Gateway, use an Azure VM (where port 445 is open), or use Azure File Sync. |
| Table Storage queries are running slowly. | Query is executing a full table scan across all partitions. | Always filter by `PartitionKey` in your queries to limit search to a single partition node. |
| Linux mount returns `mount error(13): Permission denied`. | Incorrect storage account key or special characters unescaped. | Re-copy the storage key from **Access keys** in the Azure Portal; verify no trailing spaces. |

---

## 6. Summary Checklist
- [x] Azure Table Storage is a schemaless NoSQL store using a composite key (`PartitionKey` + `RowKey`).
- [x] Ideal for storing massive, inexpensive semi-structured datasets like telemetry and logs.
- [x] Azure Files delivers enterprise cloud file shares over standard SMB and NFS protocols.
- [x] Allows concurrent multi-server mounting for legacy migrations and shared container storage.
