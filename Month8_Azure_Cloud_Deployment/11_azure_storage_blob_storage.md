# 11 - Azure Blob Storage

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `04:43:40` to `05:25:07`

---

## 1. What is an Azure Storage Account?

An **Azure Storage Account** is a secure cloud container that provides a unique namespace in Microsoft Azure for your data. Every storage account has a globally unique name that forms part of your web address:
```
https://<yourstorageaccountname>.blob.core.windows.net
```

Inside an Azure Storage Account, Microsoft provides four distinct storage services:
1. **Blob Storage:** Highly scalable object storage for unstructured data (images, videos, backups).
2. **Azure Files:** Fully managed cloud file shares accessible via standard SMB and NFS protocols.
3. **Azure Queues:** Reliable message queuing for asynchronous communication between services.
4. **Azure Tables:** Fast, serverless NoSQL key-value table storage.

In this module, we focus on the most popular and essential service: **Azure Blob Storage**.

---

## 2. What is Azure Blob Storage?

"Blob" stands for **Binary Large Object**. 

**Azure Blob Storage** is Microsoft's object storage solution for the cloud. It is specifically designed to store massive amounts of **unstructured data**—meaning data that does not conform to a rigid database schema with rows and columns.

### Real-World Use Cases:
- Serving images, logos, and documents directly to a web browser.
- Storing files for distributed access (PDF downloads, user profile avatars).
- Streaming video and audio files.
- Storing system log files, audit traces, and telemetry.
- Long-term data archiving, disaster recovery, and enterprise database backups.

---

## 3. Storage Hierarchy: How Blob Storage is Structured

Blob storage organizes data into three simple levels:

```
+-------------------------------------------------------------------------------+
|                             AZURE STORAGE ACCOUNT                             |
|                    (e.g., mystudentstorageaccount2026)                        |
|                                                                               |
|  +-------------------------------------------------------------------------+  |
|  |                         CONTAINER (FOLDER/BUCKET)                       |  |
|  |                               (e.g., "photos")                          |  |
|  |                                                                         |  |
|  |  +--------------------+  +--------------------+  +--------------------+ |  |
|  |  | Blob: profile.jpg  |  | Blob: banner.png   |  | Blob: doc.pdf      | |  |
|  |  +--------------------+  +--------------------+  +--------------------+ |  |
|  +-------------------------------------------------------------------------+  |
+-------------------------------------------------------------------------------+
```

1. **Storage Account:** The top-level Azure resource. You can have multiple containers inside one storage account.
2. **Container:** A logical grouping of blobs (similar to a folder on your computer or an S3 bucket in AWS). A storage account can contain an **unlimited** number of containers.
3. **Blob:** The actual file itself (along with its metadata). A container can store an **unlimited** number of blobs, scaling up to Petabytes.

---

## 4. The Three Types of Blobs

Azure provides three specialized types of blobs:

1. **Block Blobs (Standard & Most Common):**
   - Composed of individual blocks of data (up to 100 MB each).
   - Optimized for uploading and storing documents, images, video files, and application assets.
   - A single block blob can store up to **190.7 TB** of data!
2. **Append Blobs:**
   - Optimized for "append-only" operations where data is only ever added to the end of the file.
   - Modifying existing data or deleting blocks is not allowed.
   - **Ideal for:** Logging user actions, recording IoT sensor streams, and security audit logs.
3. **Page Blobs:**
   - Organized into a collection of 512-byte pages designed for random read and write operations.
   - **Used under the hood for:** Azure Virtual Machine Operating System and Data Disks (`.vhd` files).

---

## 5. Blob Access Tiers (Saving Cloud Costs)

Storage costs add up quickly. Azure lets you choose an **Access Tier** based on how frequently your data is accessed:

```
Frequent Access -----------------------------------------------------> Rare Access
High Storage Cost / Low Access Fee                     Low Storage Cost / High Access Fee

     [ HOT TIER ]  ---->  [ COOL TIER ]  ---->  [ COLD TIER ]  ---->  [ ARCHIVE TIER ]
    (Active Photos)       (Monthly Reports)    (Old Tax Years)       (Compliance Backups)
```

| Access Tier | Description | Minimum Storage Duration | Retrieval Speed | Best Used For |
| :--- | :--- | :--- | :--- | :--- |
| **Hot** | For data accessed or modified frequently. | None. | Immediate (milliseconds). | Active website images, product catalogs, everyday user uploads. |
| **Cool** | For data stored for at least 30 days and accessed infrequently. | 30 Days. | Immediate (milliseconds). | Short-term backups, previous month's invoices, customer support records. |
| **Cold** | For data stored for at least 90 days and accessed rarely. | 90 Days. | Immediate (milliseconds). | Audit data, quarterly logs, regulatory records accessed once or twice a year. |
| **Archive** | For offline data stored for at least 180 days with flexible latency. | 180 Days. | **Offline (Takes 1 to 15 hours to rehydrate)**. | Multi-year legal compliance records, raw historical sensor archives. Lowest storage price! |

> **Student Analogy:**
> - **Hot Tier:** The book on your study desk that you read every morning.
> - **Cool Tier:** The textbook kept on your bookshelf that you read once a month.
> - **Cold Tier:** Old notes packed into a cupboard in your bedroom.
> - **Archive Tier:** Boxes taped shut and stored in the dark attic or basement. You rarely touch them, and it takes hours to unpack them if you need something.

---

## 6. Data Redundancy: Keeping Your Data Safe

Hardware failures happen in datacenters. Azure protects your data by creating multiple copies automatically. You choose the level of redundancy:

1. **LRS (Locally Redundant Storage):**
   - Replicates your data **3 times** within a single physical datacenter.
   - Lowest cost. Protects against disk and server failures, but not against a fire/flood in that specific building.
2. **ZRS (Zone-Redundant Storage):**
   - Replicates your data across **3 separate Availability Zones** in the same region.
   - Protects against building-level disasters (e.g. power failure in one datacenter).
3. **GRS (Geo-Redundant Storage):**
   - Replicates your data 3 times in your primary region (LRS), and then asynchronously copies it 3 more times to a **secondary paired region hundreds of miles away**.
   - Guarantees data survival even if an entire region suffers a catastrophic natural disaster.
4. **GZRS (Geo-Zone-Redundant Storage):**
   - Maximum protection: 3 copies across separate Availability Zones in the primary region, plus geo-replication to a secondary region.

---

## 7. Security: How to Share Blobs Securely

How do you give someone access to download a private file without making your entire storage account public?

### The Bad Way: Storage Account Access Keys
- When you create a storage account, Azure gives you two Master Keys (`Key1` and `Key2`).
- **Never share these keys!** Anyone with a storage account key has full administrative "root" access to delete, read, or modify every container and file in your account.

### The Right Way: Shared Access Signatures (SAS)
A **Shared Access Signature (SAS)** is a secure, signed URL that provides temporary, restricted access to a specific blob or container.

When creating a SAS token, you specify:
- **Permissions:** Exactly what the user can do (e.g., `Read-only`, no delete).
- **Time Window:** An exact start time and expiry time (e.g., valid for only 30 minutes).
- **Allowed IP Addresses:** Optional restriction to only allow requests from specific IP addresses.
- **Protocol:** Enforce secure HTTPS only.

```
Example SAS URL:
https://mystorage.blob.core.windows.net/photos/cat.jpg?sp=r&st=2026-11-16T10:00:00Z&se=2026-11-16T11:00:00Z&spr=https&sig=abc12345xyz...
```

Once the expiry time passes, the link stops working automatically.

---

## 8. Hands-on Walkthrough: Creating & Sharing a Blob

In the video, M Prashant demonstrates the complete workflow:

1. **Create Storage Account:**
   - Search for **Storage accounts** -> Click **Create**.
   - Resource Group: `rg-storage-demo`.
   - Name: `azuresimplifiedstorage$RANDOM` (must be lowercase numbers and letters).
   - Redundancy: `Locally-redundant storage (LRS)`.
2. **Create Container:**
   - Go to your new Storage Account -> Select **Containers** on the left menu.
   - Click **+ Container** -> Name: `user-uploads`.
   - Public access level: Keep as **Private (no anonymous access)**.
3. **Upload a File:**
   - Click into `user-uploads` -> Click **Upload** -> Select an image from your computer (`logo.png`).
4. **Generate SAS Token & Test:**
   - Click the three dots `...` next to `logo.png` -> Click **Generate SAS**.
   - Permissions: check **Read**.
   - Expiry: Set for 1 hour from now.
   - Click **Generate SAS token and URL**.
   - Copy the **Blob SAS URL**.
   - Open an incognito/private browser window, paste the URL, and press Enter.
   - Your image displays immediately, proving secure temporary access!

---

## 9. Summary Checklist
- [x] Storage Accounts provide a centralized space for Blobs, Files, Queues, and Tables.
- [x] Blob Storage stores unstructured data (images, videos, backups) at unlimited scale.
- [x] Use Access Tiers (Hot, Cool, Cold, Archive) to dramatically reduce storage costs.
- [x] Use LRS for basic redundancy, ZRS for zone safety, and GRS for cross-region disaster recovery.
- [x] Never share root access keys; always use time-limited Shared Access Signatures (SAS) for secure sharing.
