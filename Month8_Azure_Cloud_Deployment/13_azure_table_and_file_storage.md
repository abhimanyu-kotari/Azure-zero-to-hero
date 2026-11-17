# 13 - Azure Table Storage (NoSQL) & Azure Files

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `05:57:26` to `06:22:08`

---

## Part 1: Azure Table Storage (NoSQL)

### 1. What is NoSQL?
In traditional relational databases (like MySQL or SQL Server), you must pre-define a rigid table schema with exact column names and datatypes. If Row 1 has 5 columns, every other row must adhere to that schema. Adding new columns requires running complex `ALTER TABLE` migrations.

**NoSQL (Not Only SQL)** is a non-relational, **schema-less** database model. Every record (called an **Entity**) can have completely different attributes without breaking the database.

---

### 2. What is Azure Table Storage?
**Azure Table Storage** is a highly scalable, serverless NoSQL key-value datastore included inside your Azure Storage Account.

It is designed to store massive amounts of semi-structured data (millions of records) at an **extremely low cost** (fraction of a cent per gigabyte), while delivering single-digit millisecond lookup speeds.

### Anatomy of an Entity (Record) in Table Storage:
Every row in Azure Table Storage requires three mandatory system properties:

```
+-----------------------------------------------------------------------------------------+
| PartitionKey (City) | RowKey (ID) | Timestamp             | Custom Properties...        |
+---------------------+-------------+-----------------------+-----------------------------+
| "NewYork"           | "emp_101"   | 2026-11-17T10:00:00Z  | Name="Alice", Role="Dev"    |
| "NewYork"           | "emp_102"   | 2026-11-17T10:05:00Z  | Name="Bob", Dept="QA", Yrs=3|
| "London"            | "emp_201"   | 2026-11-17T10:10:00Z  | Name="Charlie", Phone="123" |
+-----------------------------------------------------------------------------------------+
```

1. **`PartitionKey` (String):**
   - Identifies the partition (cluster) where the entity is stored.
   - Azure uses the `PartitionKey` to automatically distribute tables across thousands of storage nodes as data grows.
2. **`RowKey` (String):**
   - The unique identifier of the entity within that specific partition.
   - Together, the **`PartitionKey + RowKey`** forms the primary key of your data!
3. **`Timestamp` (DateTime):**
   - Maintained automatically by Azure to record when the entity was last modified.
4. **Custom Properties:**
   - Up to 252 custom key-value pairs per entity (supports strings, integers, booleans, GUIDs).
   - One entity can have 3 properties, while the next entity can have 20 properties!

### Best Use Cases for Table Storage:
- Storing IoT sensor telemetry and device status records.
- Storing user profile preferences and web session data.
- Maintaining customer contact books and inventory catalogs.
- High-volume logging where relational database licenses are too expensive.

---

## Part 2: Azure File Share (Azure Files)

### 1. What is Azure Files?
When you have multiple virtual machines (or developers), how do you share files between them as a standard network drive?

**Azure Files** provides fully managed cloud file shares that you can mount as a regular drive letter (`Z:\` on Windows, or `/mnt/share` on Linux) using industry-standard protocols:
- **SMB 3.0 (Server Message Block):** Used by Windows, Linux, and macOS.
- **NFS (Network File System):** Used primarily by Linux enterprise workloads.

```
                    +------------------------------------+
                    |       AZURE FILE SHARE (SMB)       |
                    |     (Managed Storage Account)      |
                    +-----------------+------------------+
                                      |
         +----------------------------+----------------------------+
         |                            |                            |
         v                            v                            v
  [ Windows VM ]                [ Linux VM ]                [ On-Premises Laptop ]
   (Mounted as Z:\)             (Mounted as /mnt/data)      (Mounted via SMB Port 445)
```

---

### 2. Why Use Azure Files?
1. **True Shared Storage:** Hundreds of virtual machines or containers can read and write to the same shared directory at the exact same time.
2. **"Lift-and-Shift" Migrations:** Legacy enterprise applications often rely on reading files from a local network share (`\\fileserver\docs`). You can migrate these apps directly to Azure without changing a single line of code!
3. **Persistent Volume for Containers:** ACI and AKS can mount an Azure File Share inside a container, ensuring data persists even if the container restarts.
4. **Azure File Sync:** Replicates your cloud Azure File Share to local on-premises Windows servers, caching hot files locally for instant access while keeping a master backup in the cloud.

---

## 3. Hands-on Walkthrough: Creating & Mounting an Azure File Share

In the video, M Prashant demonstrates creating and mounting a file share:

### Step 1: Create a File Share in Azure Storage Account
1. Open your Storage Account in the Azure Portal.
2. In the left menu under *Data storage*, click **File shares** -> Click **+ File share**.
3. Name: `shared-docs`.
4. Tier: **Transaction optimized** (or Hot).
5. Click **Create**.

### Step 2: Upload Files
1. Click into `shared-docs` -> Click **Upload**.
2. Select a text file or image and upload it.

### Step 3: Connect to a Linux Virtual Machine
1. In the file share toolbar, click **Connect**.
2. Select the **Linux** tab.
3. Azure automatically generates the exact bash command with the storage account key and mount parameters:

```bash
# 1. Create the mount directory on your Linux VM
sudo mkdir -p /mnt/azureshared

# 2. Mount the Azure File Share using CIFS/SMB
sudo mount -t cifs //azuresimplifiedstorage.file.core.windows.net/shared-docs /mnt/azureshared \
  -o vers=3.0,username=azuresimplifiedstorage,password=<YOUR_STORAGE_KEY>,dir_mode=0777,file_mode=0777,serverino

# 3. Verify the files are visible
ls -la /mnt/azureshared
```

Any file created inside `/mnt/azureshared` on your VM is immediately uploaded and synchronized to Microsoft's cloud!

---

## 4. Summary Checklist
- [x] Azure Table Storage is a fast, cheap, schemaless NoSQL key-value store.
- [x] Entities require PartitionKey + RowKey + Timestamp; custom properties can vary per row.
- [x] Azure Files provides managed SMB/NFS file shares mounted simultaneously across Windows & Linux.
- [x] Ideal for legacy lift-and-shift app migrations and container persistent storage.
