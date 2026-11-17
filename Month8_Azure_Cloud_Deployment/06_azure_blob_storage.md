# 06 - Cloud Object Storage: Azure Storage Accounts & Blob Storage

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:43:40` to `05:25:07` (Unstructured Data, Access Tiers & SAS Tokens)

---

## 1. What is an Azure Storage Account?

An **Azure Storage Account** is a secure, cloud-managed container that provides massively scalable, highly durable storage for files, objects, disks, and messages.

A single Storage Account provides four foundational storage services:

```
+---------------------------------------------------------------------------------------------------+
|                                     AZURE STORAGE ACCOUNT                                         |
+---------------------------------------------------------------------------------------------------+
|                                                                                                   |
|  1. BLOB STORAGE       2. FILE STORAGE       3. QUEUE STORAGE       4. TABLE STORAGE              |
|  [ Object Storage ]    [ Cloud SMB Shares ]  [ Async Messaging ]    [ NoSQL Key-Value Store ]     |
|  - Images, videos      - Mount as network    - Reliable async       - Semi-structured data        |
|  - Backups, logs       - drive on Windows /  - message buffering    - Schemaless records          |
|  - Big data analytics  - Linux laptops       - between services     - Ultra-low cost              |
|                                                                                                   |
+---------------------------------------------------------------------------------------------------+
```

---

## 2. Deep Dive: What is Azure Blob Storage?

**Blob** stands for **Binary Large Object**. It is Azure's object storage solution (equivalent to AWS S3 or Google Cloud Storage).

### Structured vs. Unstructured Data:
- **Structured Data:** Tabular data that fits neatly into rows and columns (e.g., User ID, Account Balance, Age in an SQL database).
- **Unstructured Data:** Data that does not conform to rigid schemas: user profile photos, video recordings, PDF contracts, audio clips, and system log files. Storing unstructured binary files inside a relational SQL database causes massive database bloat and performance degradation!

### The Three-Tier Storage Hierarchy:
```
Storage Account  --->  Container (Bucket)  --->  Blob (File)
(e.g., mystorageacct)   (e.g., product-images)    (e.g., laptop-hero.png)
```

### The Three Types of Blobs in Azure:
1. **Block Blobs:** Used for standard files, documents, photos, and video streaming. Stored as discrete blocks (up to 190.7 TB per file).
2. **Append Blobs:** Optimized for append-only operations where data is continuously written to the end of the file (ideal for live application logs).
3. **Page Blobs:** Optimized for frequent random read and write operations. Used as the underlying virtual hard disks (`.vhd` files) for Azure Virtual Machines.

---

## 3. Storage Redundancy Options (Data Durability)

To safeguard your data against disk crashes, datacenter power outages, and natural disasters, Azure replicates data across multiple locations:

| Redundancy Level | Replicas | Geographic Distribution | Resilience Guarantee |
| :--- | :--- | :--- | :--- |
| **LRS** (Locally Redundant) | **3 copies** | Stored within a **single physical datacenter** in one region. | Protects against server disk/rack failure. Vulnerable to datacenter fires. |
| **ZRS** (Zone-Redundant) | **3 copies** | Replicated across **3 distinct Availability Zones** in the same region. | Protects against entire datacenter outages. |
| **GRS** (Geo-Redundant) | **6 copies** | **3 copies in primary region (LRS) + 3 copies replicated to a paired region hundreds of miles away!** | Protects against regional natural disasters (floods, earthquakes). |
| **GZRS** (Geo-Zone-Redundant)| **6 copies** | **3 copies across 3 zones in primary region + 3 copies in secondary paired region.** | Maximum enterprise-grade protection. |

---

## 4. Understanding Blob Access Tiers (Cost Optimization)

To help companies minimize cloud storage bills, Azure provides four access tiers based on how frequently files are accessed:

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

| Tier | Ideal Use Case | Minimum Storage Duration | Retrieval Latency |
| :--- | :--- | :--- | :--- |
| **Hot** | Data in active, daily use (e.g., website product photos). | None | Immediate (Milliseconds) |
| **Cool** | Data accessed less than once a month (e.g., monthly invoices). | **30 days** | Immediate (Milliseconds) |
| **Cold** | Rarely accessed data (e.g., quarterly financial backups). | **90 days** | Immediate (Milliseconds) |
| **Archive**| Long-term compliance backups, legal records, audit logs. | **180 days** | **Requires Rehydration** (Takes 1 to 15 hours!) |

> [!CAUTION]
> If you delete or transition a file out of the Archive tier before the 180-day minimum, Azure charges an early deletion fee.

---

## 5. Security & Access: Access Keys vs. SAS Tokens vs. Entra ID

```
+-------------------------------------------------------------------------+
|                  STORAGE AUTHENTICATION METHODS                         |
+-------------------------------------------------------------------------+
|  1. ACCESS KEYS (Root Master Keys):                                     |
|     - Complete administrative control (Read, Write, Delete, Reconfigure)|
|     - NEVER expose in client apps or commit to GitHub repositories!     |
|                                                                         |
|  2. SHARED ACCESS SIGNATURES (SAS):                                     |
|     - Time-limited, permission-restricted signed URLs.                  |
|     - Example: "Grant User A READ-ONLY access to invoice.pdf for 2 hrs" |
|                                                                         |
|  3. MICROSOFT ENTRA ID (Azure AD RBAC - Best Practice):                 |
|     - Identity-based RBAC roles (Storage Blob Data Reader/Contributor). |
|     - Zero passwords or tokens stored in source code.                   |
+-------------------------------------------------------------------------+
```

---

## 6. Hands-on Lab: Containers, Uploads & SAS Tokens

### Step 1: Create a Storage Account in Azure Portal
1. Search for **Storage accounts** in the Azure Portal $\rightarrow$ click **+ Create**.
2. **Basics Tab:**
   - **Subscription:** Select your subscription.
   - **Resource group:** `rg-storage-demo`.
   - **Storage account name:** `mystoragedemo12345` *(Must be globally unique, 3-24 characters, lowercase alphanumeric only!)*.
   - **Region:** `East US`.
   - **Performance:** **Standard** (general purpose v2).
   - **Redundancy:** **Locally-redundant storage (LRS)** (cheapest for practice).
3. Click **Review + create** $\rightarrow$ click **Create**.

---

### Step 2: Create a Private Blob Container
1. Open your new Storage Account.
2. In the left navigation menu under **Data storage**, click **Containers**.
3. Click **+ Container**:
   - **Name:** `secure-documents`.
   - **Anonymous access level:** **Private (no anonymous access)**.
   - Click **Create**.

---

### Step 3: Upload a File & Test Private Access
1. Click into `secure-documents`.
2. Click **Upload** $\rightarrow$ select an image from your computer (e.g., `architecture.png`).
3. Click **Upload**.
4. Click on the uploaded file and copy its **URL** (e.g., `https://mystoragedemo12345.blob.core.windows.net/secure-documents/architecture.png`).
5. Open an Incognito browser tab and paste the URL.
   - **Result:** `<Error><Code>ResourceNotFound</Code><Message>The specified resource does not exist.</Message></Error>`
   - *Why?* Because the container is private, protecting your file from unauthorized internet access!

---

### Step 4: Generate a Shared Access Signature (SAS) Token
1. In the file overview window, click **Generate SAS**:
   - **Signing method:** Account key.
   - **Permissions:** Check **Read** only (Uncheck Write, Delete, List).
   - **Start and expiry time:** Set expiry to **1 hour from now**.
   - **Allowed protocols:** `HTTPS only`.
2. Click **Generate SAS token and URL**.
3. Copy the **Blob SAS URL** (Notice the secure cryptographic token appended to the URL: `?sp=r&st=2026-11-17...&sig=XYZ`).
4. Paste the SAS URL into your browser:
   - **Result:** The image displays immediately!
   - In 60 minutes, the link permanently expires, safeguarding your storage asset!

---

## 7. High-Speed Transfers: AzCopy CLI

**AzCopy** is Microsoft's high-speed, parallel command-line data movement utility:

```bash
# 1. High-speed upload from local directory to Azure Blob Container
azcopy copy "/local/photos/*" "https://mystoragedemo12345.blob.core.windows.net/secure-documents?<SAS_TOKEN>" --recursive

# 2. Synchronize local directory with Azure Blob Container (Incremental copy)
azcopy sync "/local/backup" "https://mystoragedemo12345.blob.core.windows.net/backups?<SAS_TOKEN>"
```

---

## 8. Troubleshooting Common Storage Issues

| Problem | Root Cause | Solution |
| :--- | :--- | :--- |
| Storage account name rejected during creation. | Name must be globally unique across all Microsoft Azure customers worldwide. | Use lowercase letters and numbers only; append unique numbers (e.g., `storeabhimanyu2026`). |
| Public URL returns `PublicAccessNotPermitted`. | Storage account level disallows anonymous access. | Enable **Allow Blob anonymous access** in Storage Account Configuration settings, or use SAS tokens. |
| SAS URL returns `AuthenticationFailed`. | Token expired or system clock skew. | Verify expiry timestamp on SAS token. Ensure your local machine clock is synchronized with UTC. |

---

## 9. Summary Checklist
- [x] Azure Storage Accounts provide Blobs, Files, Queues, and Tables in a unified service.
- [x] Blob storage is designed for unstructured object data (Block, Append, and Page blobs).
- [x] Redundancy options (LRS, ZRS, GRS) protect data against hardware crashes and regional disasters.
- [x] Access tiers (Hot, Cool, Cold, Archive) allow aggressive cost optimization based on file access frequency.
- [x] SAS tokens provide granular, time-bound, secure URL access to private storage assets.
