# 11 - Cloud Object Storage: Azure Storage & Blob Storage

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:43:40` to `05:25:07`

---

## 1. What is an Azure Storage Account?

An **Azure Storage Account** is a secure, cloud-managed repository that provides massively scalable, highly available object, file, disk, queue, and table storage.

A single Storage Account provides four foundational services:

```
+-------------------------------------------------------------------------+
|                          AZURE STORAGE ACCOUNT                          |
+-------------------------------------------------------------------------+
|                                                                         |
|  1. BLOB STORAGE       2. FILE STORAGE       3. QUEUE STORAGE  4. TABLES|
|  [ Object Storage ]    [ Cloud SMB Shares ]  [ Async Messaging][ NoSQL ]|
|  - Images, videos      - Mount as network    - Reliable async  - Key/Val|
|  - Backups, logs       - drive on Windows/   - message queues  - Schema-|
|  - Big data analytics  - Linux laptops       - between apps    - less   |
|                                                                         |
+-------------------------------------------------------------------------+
```

---

## 2. Deep Dive: What is Azure Blob Storage?

**Blob** stands for **Binary Large Object**. It is Azure's unstructured data store (similar to AWS S3 or Google Cloud Storage). Unstructured data means data that does not fit into rigid rows and columns like a SQL database.

### Organization Hierarchy:
```
Storage Account  --->  Container (Bucket)  --->  Blob (File)
(e.g., mystorageacct)   (e.g., product-images)    (e.g., logo.png)
```

### Types of Blobs in Azure:
1. **Block Blobs:** Used for standard files, documents, photos, and video streaming. Composed of discrete blocks of data (up to 190.7 TB total size).
2. **Append Blobs:** Optimized for append-only operations where data is continuously written to the end of the file (ideal for real-time application logs).
3. **Page Blobs:** Designed for frequent random read and write operations. Used as the underlying virtual hard disks (`.vhd` files) for Azure Virtual Machines.

---

## 3. Storage Redundancy Options (Replication)

Azure ensures your data survives hardware crashes, fires, and natural disasters through automated data replication:

| Redundancy Level | Copies | Geographic Distribution | Resilience |
| :--- | :--- | :--- | :--- |
| **LRS** (Locally Redundant Storage) | **3 copies** | Stored within a **single physical datacenter** in one region. | Protects against server rack/disk failure. Vulnerable to datacenter fires. |
| **ZRS** (Zone-Redundant Storage) | **3 copies** | Replicated across **3 distinct Availability Zones** in the same region. | Protects against entire datacenter outages. |
| **GRS** (Geo-Redundant Storage) | **6 copies** | **3 copies in primary region (LRS) + 3 copies replicated to a paired region hundreds of miles away!** | Protects against massive regional disasters (hurricanes, earthquakes). |
| **GZRS** (Geo-Zone-Redundant Storage) | **6 copies** | **3 copies across 3 zones in primary region + 3 copies in secondary paired region.** | Maximum enterprise-grade protection. |

---

## 4. Understanding Blob Access Tiers (Cost Optimization)

To help organizations minimize storage spend, Azure provides four access tiers:

```
+--------------------------------------------------------------------------+
|                         BLOB STORAGE ACCESS TIERS                        |
+--------------------------------------------------------------------------+
|                      Storage Cost (per GB/month)                         |
|   Hot ($$$)  ----->  Cool ($$)  ----->  Cold ($)  ----->  Archive (cents)|
|                                                                          |
|                      Data Access Cost (per GB read)                      |
|   Hot (Lowest) <---  Cool ($)   <-----  Cold ($$) <-----  Archive ($$$)  |
|                                                                          |
|                      Retrieval Latency                                   |
|   Milliseconds  <--  Milliseconds <---  Milliseconds <--  Hours (Wait!)  |
+--------------------------------------------------------------------------+
```

| Tier | Usage Scenario | Minimum Stored Duration | Retrieval Time |
| :--- | :--- | :--- | :--- |
| **Hot** | Data in active use (e.g., homepage images, active user files). | None | Immediate (Milliseconds) |
| **Cool** | Data accessed less than once a month (e.g., monthly invoices, staging data). | **30 days** | Immediate (Milliseconds) |
| **Cold** | Rarely accessed data (e.g., historical snapshots). | **90 days** | Immediate (Milliseconds) |
| **Archive**| Long-term compliance backups, legal records, audit trails. | **180 days** | **Requires Rehydration** (Takes 1 to 15 hours!) |

> [!CAUTION]
> If you delete or transition a file out of the Archive tier before the 180-day minimum, Azure charges an early deletion fee.

---

## 5. Security & Access: Access Keys vs. SAS Tokens vs. Entra ID

How do applications and users authenticate with Blob Storage?

```
+-------------------------------------------------------------------------+
|                  STORAGE AUTHENTICATION METHODS                         |
+-------------------------------------------------------------------------+
|  1. ACCESS KEYS (Root Keys):                                            |
|     - Complete administrative control (Read, Write, Delete, Reconfigure)|
|     - NEVER expose in client applications or commit to Git!             |
|                                                                         |
|  2. SHARED ACCESS SIGNATURES (SAS):                                     |
|     - Time-limited, permission-restricted signed URLs.                  |
|     - Example: "Grant User A READ-ONLY access to invoice.pdf for 2 hrs" |
|                                                                         |
|  3. MICROSOFT ENTRA ID (Azure AD - Best Practice):                      |
|     - Identity-based RBAC roles (Storage Blob Data Reader/Contributor). |
|     - Zero secrets or tokens stored in source code.                     |
+-------------------------------------------------------------------------+
```

---

## 6. Hands-on Lab: Creating Containers, Uploading Blobs & Generating SAS Tokens

### Step 1: Create a Storage Account
1. Open the [Azure Portal](https://portal.azure.com).
2. Search for **Storage accounts** and click **+ Create**.
3. **Basics Tab:**
   - **Subscription:** Select your subscription.
   - **Resource group:** `rg-storage-demo`.
   - **Storage account name:** `mystoragedemo12345` *(Must be globally unique, 3-24 characters, lowercase alphanumeric only!)*.
   - **Region:** `East US`.
   - **Performance:** **Standard** (general purpose v2).
   - **Redundancy:** **Locally-redundant storage (LRS)** (cheapest for learning).
4. Click **Review + create** -> **Create**.

---

### Step 2: Create a Blob Container
1. Open your new Storage Account.
2. In the left menu under **Data storage**, click **Containers**.
3. Click **+ Container**:
   - **Name:** `public-assets`.
   - **Anonymous access level:** **Private (no anonymous access)** (Best practice security).
   - Click **Create**.

---

### Step 3: Upload a File
1. Click on your `public-assets` container.
2. Click **Upload** -> select an image from your computer (e.g., `cloud-architecture.png`).
3. Click **Upload**.
4. Click the uploaded file and copy its **URL** (e.g., `https://mystoragedemo12345.blob.core.windows.net/public-assets/cloud-architecture.png`).
5. Paste the URL into a new browser window.
   - **Result:** `<Error><Code>ResourceNotFound</Code><Message>The specified resource does not exist.</Message></Error>`
   - *Why?* Because the container is private, preventing unauthorized internet strangers from viewing your files!

---

### Step 4: Generate a Shared Access Signature (SAS) Token
1. In the file overview window, click **Generate SAS**:
   - **Signing method:** Account key.
   - **Permissions:** Check **Read** only (Uncheck Write, Delete, List).
   - **Start and expiry time:** Set expiry to **1 hour from now**.
   - **Allowed protocols:** `HTTPS only`.
2. Click **Generate SAS token and URL**.
3. Copy the **Blob SAS URL** (Notice the cryptographic query string appended: `?sp=r&st=2026-11-17...&sig=XYZ`).
4. Paste the SAS URL into your browser:
   - **Result:** The image displays instantly!
   - In 1 hour, that link permanently expires, protecting your data.

---

## 7. Azure CLI & AzCopy Cheat Sheet

```bash
# 1. Create a storage account via CLI
az storage account create \
  --name storagedemo$RANDOM \
  --resource-group rg-storage-demo \
  --location eastus \
  --sku Standard_LRS

# 2. Upload a file using Azure CLI
az storage blob upload \
  --account-name storagedemo1234 \
  --container-name public-assets \
  --name myphoto.png \
  --file ./myphoto.png \
  --auth-mode login

# 3. High-speed file transfer using AzCopy utility
# Download AzCopy and transfer an entire directory in parallel:
azcopy copy "/local/photos/*" "https://mystorage.blob.core.windows.net/public-assets?<SAS_TOKEN>" --recursive
```

---

## 8. Summary Checklist
- [x] Azure Storage Accounts provide Blobs, Files, Queues, and Tables in a unified service.
- [x] Blob storage stores unstructured data (Block Blobs, Append Blobs, Page Blobs).
- [x] Redundancy options (LRS, ZRS, GRS) safeguard data against hardware crashes and regional disasters.
- [x] Access tiers (Hot, Cool, Cold, Archive) allow aggressive cost optimization based on file access frequency.
- [x] SAS tokens provide granular, time-bound, secure URL access to private storage assets.
