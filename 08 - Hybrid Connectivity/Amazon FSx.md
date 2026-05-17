---
tags:
  - aws/fsx
  - storage
  - file-system
  - hybrid
category: Hybrid Connectivity
---

# 📁 Amazon FSx

> [!INFO] Definition
> **Amazon FSx** provides fully managed, highly reliable, and performant file storage that is accessible over the industry-standard Server Message Block (SMB) protocol and high-performance computing (HPC) interfaces. It makes it easy to launch and run popular file systems in the cloud.

AWS offers several versions of FSx, but for the AWS Certified Solutions Architect Associate exam, the two main focuses are **FSx for Windows File Server** and **FSx for Lustre**.

---

## 🪟 FSx for Windows File Server

FSx for Windows provides fully managed, native Windows file servers and file shares. It is built on Windows Server and designed to integrate seamlessly into Windows environments.

![[FSX-Windows-1.png]]

### Key Features
* **Native Windows Compatibility**: Supports the **SMB protocol**, Windows NTFS file system, Active Directory (AD) integration, and Distributed File System (DFS).
* **Active Directory Integration**: Uses AD for its user store and permission model (can be AWS Managed Microsoft AD or your self-managed on-premises AD).
* **Accessibility**: Accessed via VPC natively; supports Peering, VPN, and Direct Connect for hybrid setups.
* **Resilience**: Can be deployed in **Single-AZ** or **Multi-AZ** configurations.
* **Data Deduplication**: Supported, reducing storage costs.
* **Encryption**: KMS at-rest encryption and enforced in-transit encryption.

### Volume Shadow Copy Service (VSS)
**VSS (User-driven restores)** allows users to see multiple file versions and initiate restores directly from the client side.
* If using an FSx share from a Windows environment, users can right-click a file or folder, select "Restore previous versions," and perform file-level restores **without IT/SysAdmin intervention**.

> [!TIP] Exam Rule: EFS vs. FSx for Windows
> If you need shared storage for **Linux** instances → **Amazon EFS**.
> If you need shared storage for **Windows** instances (SMB/AD integration) → **Amazon FSx for Windows**.

---

## 🚀 FSx for Lustre

FSx for Lustre is a file system designed specifically for extreme high-performance computing (HPC) workloads, such as machine learning, big data analytics, and financial modeling.

![[FSX-Lustre-1.png]]

### Key Features
* **Extreme Performance**: Provides **sub-millisecond latency** and hundreds of GB/s throughput.
* **Linux Centric**: Supports Linux-based instances running AWS and uses **POSIX-style permissions**.
* **S3 Integration**: You can associate a Lustre file system to an S3 bucket repository. Data in the S3 bucket becomes immediately visible in the file system.
  * Data is **lazy-loaded** into the file system only when accessed.
  * *Note: No built-in auto-sync back to S3; you must run the `hsm_archive` command to sync changes.*

### Deployment Types
1. **Scratch**: Optimised for the absolute best performance for **short-term workloads**. Does **not** provide high availability (HA), resilience, or replication. If the file server fails, data is lost. Base 200MB/s per TiB of storage.
2. **Persistent**: Great for longer-term storage and HA. Includes self-healing and replication **within a single AZ** (no multi-AZ option). Offers 50MB/s, 100MB/s, or 200MB/s per TiB baselines with a credit-based burst system (up to 1300MB/s per TiB).

### Architecture & Storage Mechanics
![[FSX-Lustre-2.png]]

Lustre splits data up when storing it to disk:
* **Metadata** (file names, timestamps, permissions) are stored on **Metadata Targets (MDTs)**.
* **Data** (objects) are stored on **Object Storage Targets (OSTs)**.
* Deployed within a client-managed VPC (accessed via a single ENI in your VPC, as it is a single-AZ service). Storage servers include an in-memory cache for faster access.

---

## 🎯 Exam PowerUP: FSx for Windows vs. FSx for Lustre

> [!IMPORTANT]
> * **Windows / SMB / Active Directory / NTFS** → **FSx for Windows File Server**.
> * **Linux / HPC / Machine Learning / Big Data** → **FSx for Lustre**.
> * **Need to process data sitting in S3 at sub-millisecond speeds** → **FSx for Lustre**.
> * **Need users to self-restore files (VSS)** → **FSx for Windows File Server**.
