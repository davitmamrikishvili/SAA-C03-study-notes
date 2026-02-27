---
tags:
  - aws/ebs
  - storage
category: Storage
---

# EBS - Elastic Block Store

> [!INFO] Definition
> Amazon EBS is a high-performance **block storage** service designed for use with Amazon EC2 for both throughput and transaction-intensive workloads.

## Core Properties

*   **Block Storage**: Volumes are presented to the OS as a collection of blocks. They are **mountable** and **bootable**.
*   **Network Attached**: Delivered over the network via the AWS backbone.
    *  **Independence**: Because they are separate from the physical EC2 host, **EBS volumes "follow" the instance** even if it is stopped and restarted on a completely different hardware host.
*   **Detachable & Reattachable**: A volume can be easily detached from one instance and attached to another within the same AZ.
*   **Resiliency**: Replicated within an Availability Zone to protect against hardware failure.
*   **Persistent**: Data remains even after the EC2 instance is stopped or terminated.
*   **Encryption**: Storage can be encrypted using **AWS KMS** with zero performance impact.

![[EBS.png]]

---

## 💰 Pricing & Billing
*   **Provisioned Capacity**: You are billed per **GB-month** for the amount of storage you provision, regardless of how much data you actually store.
*   **Performance (Optional)**: Higher-tier volumes (like `gp3` or `io2`) may have separate charges for provisioned **IOPS** and **Throughput**.

---

## 🏗️ EBS vs. Instance Store

| Property | Instance Store | EBS |
| :--- | :--- | :--- |
| **Type** | Ephemeral (Temporary) | Persistent (Permanent) |
| **Location** | Physically attached to Host | Network Attached |
| **Persistence** | Lost on Host failure/Stop | Persists past Instance lifetime |
| **Speed** | Extremely High (Lowest Latency) | High (Varies by type) |

---

## 📊 Storage Types Comparison

| Type | Structure | Mountable? | Bootable? |
| :--- | :--- | :--- | :--- |
| **Block (EBS)** | Flat collection of blocks (Block IDs) | ✅ Yes | ✅ Yes |
| **File (EFS)** | Hierarchical file system (NFS) | ✅ Yes | ❌ No |
| **Object (S3)** | Flat collection of objects (Keys/Values) | ❌ No | ❌ No |

---

## 🚀 Storage Performance
Performance is measured by how much data can be moved and how fast.

> [!TIP] The Formula
> **IO Size × IOPS = Throughput**

*   **IO Size**: The size of each individual block being read or written.
*   **IOPS**: Input/Output Operations Per Second.
*   **Throughput**: The total volume of data moved per second.

---

## 🚧 Key Limitations

### 📍 AZ Affinity
> [!ERROR] Critical Exam Nugget
> An EBS volume is **locked to a specific Availability Zone**. To move it to another AZ, you MUST take a **Snapshot** and restore it in the new AZ.

### 🔗 Attachment Limits
*   **Single-Attach**: Most volumes can only be attached to one instance at a time.
*   **Multi-Attach**: Supported by specific volume types (like `io1` and `io2`) on Nitro-based instances, allowing one volume to be shared between multiple instances.

---

## 💾 General Purpose SSD (GP2)

> [!INFO] Definition
> **GP2** is the legacy General Purpose SSD tier. It uses a "Burst Bucket" model where performance is directly tied to the size of the volume.

### 🪣 How IO Credits Work (The Burst Bucket)
To understand GP2, imagine a "Bucket" that holds **IO Credits**.
*   **Credits**: 1 Credit allows for 1 I/O operation (up to 256 KiB, but measured in **16 KiB units** for credit consumption).
*   **The Bucket Size**: Every volume starts with a full bucket of **5.4 million credits**.
*   **Earning (Baseline)**: The bucket "fills up" at a rate called **Baseline Performance**.
    *   **Rate**: 3 IOPS per GiB (Min 100, Max 16,000).
*   **Spending (Bursting)**: When your app needs more speed than the baseline, it "bursts" to **3,000 IOPS** by draining credits from the bucket.
*   **Empty Bucket**: If the bucket runs dry, your performance drops immediately to the **Baseline** rate until you stop using the disk enough for the bucket to refill.

### 📊 Performance Scaling
| Volume Size | Baseline Performance (Earn Rate) | Burst Performance (Spend Rate) |
| :--- | :--- | :--- |
| **< 33.33 GiB** | 100 IOPS (Hard Floor) | 3,000 IOPS |
| **33.34 - 1,000 GiB** | 3 IOPS × Size (e.g., 100 GiB = 300 IOPS) | 3,000 IOPS |
| **> 1,000 GiB** | Baseline is already $\ge$ 3,000 IOPS | No "Bursting" (Baseline is higher) |

> [!TIP] Exam Nugget: Use Cases
> *   **GP2** is excellent for **Boot Volumes**, Dev/Test environments, and low-latency interactive applications because the initial 5.4M credits allow for a 30-minute burst to 3,000 IOPS—perfect for OS startup and software installation.
*   **Elastic Volumes**: You can change a volume from **GP2 to GP3** (the modern standard) while it's in use with zero downtime.

---

## ⚡ General Purpose SSD (GP3)

> [!INFO] Why move to GP3?
> GP3 is the evolution of GP2. Its biggest advantage is that it **decouples storage from performance**.

*   **Fixed Baseline**: Every GP3 volume gets a baseline of **3,000 IOPS** and **125 MB/s** throughput for free, regardless of size (even at 1 GiB!).
*   **Provisioned Performance**: You can pay to increase IOPS (up to 16,000) or Throughput (up to 1,000 MB/s) without increasing the disk size.
*   **Cost**: Generally **20% cheaper** per GiB than GP2.
*   **The Lesson**: For almost any workload, **GP3 is the preferred choice** over GP2 today.

> [!TIP] Exam Nugget: Use Cases
> *   **GP3** is excellent for **Virtual desktops**, medium sized single instance databases, such as MSSQL Server and Oracle DB, low-latency interactive apps, **Boot Volumes** and Dev/Test environments.

---

## 🏎️ Provisioned IOPS SSD (io1/io2)

> [!INFO] Definition
> **Provisioned IOPS** volumes are designed for storage-intensive applications that require the highest performance, consistent low latency, and high durability.

*   **Performance Control**: Unlike GP2/GP3, you specify exactly how many IOPS you need. IOPS are **independent of storage size**.
*   **Latency & Jitter**: Provides consistent **sub-millisecond latency** and very low jitter, which is critical for high-performance database workloads.
*   **Durability**:
    - **io1**: Designed for 99.8% - 99.9% durability.
    - **io2**: Designed for **99.999%** durability (100x more resilient than `io1`).
*   **io2 Block Express**: The highest-selling performance tier in the cloud. It runs on AWS Nitro System hardware and supports up to **256,000 IOPS** and **64 TiB** volume sizes.

> [!TIP] Exam Nugget: Use Cases
> *   **I/O Intensive Databases**: NoSQL (Cassandra, MongoDB) and Relational (Oracle, SQL Server, MySQL).
> *   **Consistent Performance**: Choose `io2` over `gp3` when your application cannot tolerate even minor performance spikes or latency variations.
> *   **Massive Volumes**: If you need a single volume larger than 16 TiB, **io2 Block Express** is your only choice (up to **64 TiB**).

---

## 💿 Hard Disk Drive (HDD) Volumes

> [!INFO] Definition
> HDD-based volumes are designed for large, sequential data workloads where **Throughput (MB/s)** is more important than IOPS.

### ⚡ Throughput Optimized HDD (st1)
*   **Performance**: Focused on high-speed sequential processing.
*   **Throughput Scaling**:
    *   **Base**: 40 MB/s per TiB.
    *   **Burst**: 250 MB/s per TiB (Max 500 MB/s total).
*   **Size**: 125 GiB to 16 TiB.
*   **Use Cases**: Big Data, Data Warehouses, Log Processing, Kafka, and MapReduce.

### ❄️ Cold HDD (sc1)
*   **Performance**: The lowest cost EBS volume type, designed for infrequent access.
*   **Throughput Scaling**:
    *   **Base**: 12 MB/s per TiB.
    *   **Burst**: 80 MB/s per TiB (Max 250 MB/s total).
*   **Size**: 125 GiB to 16 TiB.
*   **Use Cases**: Infrequently accessed data, large file archives, and maximum economy workloads where performance is secondary.

> [!ERROR] Critical Exam Nugget: The HDD "No Boot" Rule
> **HDD-based volumes (`st1` and `sc1`) CANNOT be used as Boot Volumes.** If an exam question asks for a low-cost boot volume, the answer must be an SSD-based volume (typically **GP3**).

---

## 📊 EBS Volume Types Comparison

> [!INFO] Choosing the Right Volume
> AWS offers two main categories of EBS: **SSD** (for small, random I/O like databases) and **HDD** (for large, sequential I/O like streaming).

| Volume Type | Technology | Min Size | Max Size | Max IOPS / Throughput | Best For... |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **GP3** | SSD | 1 GiB | 16 TiB | 16,000 / 1,000 MiB/s | **General Purpose (Recommended)**. Decoupled performance/storage. |
| **GP2** | SSD | 1 GiB | 16 TiB | 16,000 / 250 MiB/s | Legacy General Purpose. Performance tied to size. |
| **io2 Block Express** | SSD | 4 GiB | 64 TiB | **256,000** / 4,000 MiB/s | **Extreme performance**. SAP HANA, Oracle, SAS Analytics. |
| **io2** | SSD | 4 GiB | 16 TiB | 64,000 / 1,000 MiB/s | **Sub-millisecond latency**. Critical databases & apps. |
| **io1** | SSD | 4 GiB | 16 TiB | 64,000 / 1,000 MiB/s | Legacy Provisioned IOPS. |
| **st1** | HDD | 125 GiB | 16 TiB | 500 / 500 MiB/s | **Streaming/Big Data**. MapReduce, Kafka, Log processing. |
| **sc1** | HDD | 125 GiB | 16 TiB | 250 / 250 MiB/s | **Cold Data**. Infrequently accessed, large workloads (Archives). |

---

## ⚡ Instance Store Volumes

> [!INFO] Definition
> **Instance Store** provides temporary block-level storage for your instance. This storage is located on disks that are **physically attached** to the host computer.

*   **Ephemeral Nature**: Data in an instance store persists only during the lifetime of its associated instance.
    *   **Survives**: OS Reboots.
    *   **Lost on**: Instance Stop, Hibernate, Termination, or Hardware Failure.
*   **Attachment**: **Must be defined at launch.** You cannot add instance store volumes to an existing instance.
*   **Performance**: provides the **highest possible IOPS and throughput** in AWS because it avoids network latency.
*   **Cost**: Included in the instance's hourly price (no extra storage bill).

---

## 🏗️ EBS vs. Instance Store: Decision Guide

| If you need...                                  |   Use **EBS**   | Use **Instance Store** |
| :---------------------------------------------- | :-------------: | :--------------------: |
| **Data Persistence** (Survival past Stop/Start) |        ✅        |           ❌            |
| **Data Resilience** (Built-in replication)      |        ✅        |           ❌            |
| **High Performance** (Lowest Latency)           |    ⚠️ (High)    |      ✅ (Highest)       |
| **Storage Flexibility** (Detach/Reattach)       |        ✅        |           ❌            |
| **Cost Efficiency**                             | ⚠️ (Pay per GB) |      ✅ (Included)      |

> [!TIP] The "It Depends" Scenarios
> If your application has **built-in replication** (like a MongoDB or Cassandra cluster), you might choose **Instance Store** for the speed, knowing that the *application* handles data durability across other nodes.

---

## 🎯 EBS Selection Logic

*   **Lowest Cost?** $\to$ **SC1** (Cold HDD).
*   **Throughput/Streaming?** $\to$ **ST1** (Throughput Optimized).
*   **Boot Volume?** $\to$ **GP3** (Recommended) or **io2**. (Never HDD-based).
*   **General Purpose?** $\to$ **GP3** (Up to 16,000 IOPS).
*   **Extreme Performance?** $\to$ **io2 Block Express** (Up to 256,000 IOPS).
*   **Need > 256k IOPS?** $\to$ **Instance Store**.

---

### 🛡️ Achieving Extreme Performance: RAID 0 with EBS

> [!INFO] The Concept
> Even if a single EBS volume (like io2) isn't fast enough, you can use the Operating System (via software RAID) to **stripe** multiple EBS volumes together.

*   **RAID 0 (Striping)**: Data is split across multiple volumes. This allows you to combine the IOPS and throughput of several volumes into one logical drive.
*   **Performance Limit**: When using RAID 0, your performance is no longer limited by the single volume, but by the **Max EBS Bandwidth of the Instance Type** itself (e.g., a `c5.4xlarge` has a specific cap on how much data it can talk to the EBS network).
*   **Trade-off**: RAID 0 provides no redundancy. If **one** EBS volume fails, the entire RAID set is lost. (Note: EBS volumes are already internally replicated, so this is safer than RAID 0 on physical hardware, but still a risk).

---
*Next Topic: EBS Snapshots & Data Lifecycle Manager (DLM)*


