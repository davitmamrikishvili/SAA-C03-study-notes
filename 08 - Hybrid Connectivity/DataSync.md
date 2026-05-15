---
tags:
  - aws/datasync
  - migration
  - storage
  - hybrid
category: Hybrid Connectivity
---

# 🔄 AWS DataSync

> [!INFO] Definition
> **AWS DataSync** is an online data transfer service that moves data **TO and FROM** AWS at massive scale. It preserves file metadata, validates data integrity, and operates over the internet or Direct Connect — making it ideal for migrations, recurring data processing, and disaster recovery.

---

## 🤔 Why DataSync? (vs. Snow Family)

|                     | **DataSync**                          | **Snow Family**                |
| :------------------ | :------------------------------------ | :----------------------------- |
| **Transfer method** | Online (network-based)                | Offline (physical devices)     |
| **Speed**           | Up to 10 Gbps per agent (~100 TB/day) | Weeks for shipping + ingestion |
| **Use case**        | Ongoing sync, recurring transfers, DR | One-time massive migrations    |
| **Connectivity**    | Requires internet or Direct Connect   | No network needed              |
| **Metadata**        | Preserved (permissions, timestamps)   | Preserved                      |

> [!TIP] Rule of Thumb
> If you can transfer the data over the network in under a week, use **DataSync**. If it'll take longer (or saturate your link), use the **Snow Family**.

---

## ⚙️ How DataSync Works

![[Datasync-1.png]]

DataSync uses a **three-component architecture**:

### 1. Agent
- Software deployed **on-premises** (VMware ESXi, Hyper-V, or KVM).
- Reads/writes to on-prem storage using **NFS** (Network File System) or **SMB** (Server Message Block).
- Each agent can sustain **up to 10 Gbps** of throughput.
- You can deploy multiple agents for more bandwidth.

### 2. Task
- A **job definition** that defines:
  - **What** to sync (source and destination locations)
  - **How quickly** (bandwidth limits, schedule)
  - **Filtering** (include/exclude patterns, file age)
- Each task can handle **up to 50 million files**.
- Supports **incremental transfers** — only changed files are copied after the initial sync.

### 3. Location
- Every task has **two locations**: FROM and TO.
- **On-premises**: NFS, SMB
- **AWS destinations**: Amazon S3, Amazon EFS, Amazon FSx for Windows File Server, Amazon FSx for Lustre, Amazon FSx for NetApp ONTAP, Amazon FSx for OpenZFS
- Locations are **bi-directional** — you can sync FROM AWS back TO on-premises.

> [!INFO] How Tasks Execute
> 1. Agent scans the source location.
> 2. Compares with destination (checksum-based).
> 3. Transfers only **new or changed** data.
> 4. Validates integrity after transfer.
> 5. Preserves metadata (permissions, ownership, timestamps).

---

## ✨ Key Features

| Feature                   | Detail                                                         |
| :------------------------ | :------------------------------------------------------------- |
| **Throughput**            | Up to 10 Gbps per agent (~100 TB/day)                          |
| **Scale**                 | Up to 50 million files per task                                |
| **Bandwidth Limiters**    | Throttle transfer speed to avoid saturating your internet link |
| **Incremental Transfers** | After initial sync, only changed data is transferred           |
| **Scheduling**            | Recurring jobs (hourly, daily, weekly) for ongoing sync        |
| **Compression**           | Data is compressed in transit to reduce bandwidth usage        |
| **Encryption**            | TLS in-transit; KMS can encrypt data at rest in S3/EFS/FSx     |
| **Data Validation**       | Built-in checksum verification on every file transferred       |
| **Auto Recovery**         | Automatically retries on transient network errors              |
| **Metadata Preservation** | Permissions, ownership, timestamps are preserved               |
| **Pricing**               | Pay per GB transferred — no upfront costs                      |

---

## 🎯 Common Use Cases

- **Data Migration** — Move on-premises NAS/SAN data to AWS storage services (S3, EFS, FSx).
- **Hybrid Data Processing** — Sync on-prem data to AWS for processing (analytics, ML), then sync results back.
- **Archival** — Move cold data to cost-effective S3 storage classes (Glacier, Deep Archive) via S3 lifecycle policies after transfer.
- **Disaster Recovery / Business Continuity** — Continuously replicate on-prem file data to AWS storage for DR.

---

## 🧠 Exam Pointers

- **DataSync = online, Snow Family = offline.** If the question mentions "over the internet" or "Direct Connect" and "regular sync," it's DataSync.
- **Metadata is preserved** (permissions, timestamps) — this is a frequently tested differentiator vs. basic S3 CLI copies.
- **NFS and SMB** are the protocols the agent uses to talk to on-prem storage — not S3 API, not iSCSI.
- **Incremental sync** is built-in — you don't need to script rsync or manage deltas yourself.
- **Bandwidth limiters** protect your production network from saturation during business hours.
- **Agent runs on-prem** — it's a VMware/Hyper-V/KVM virtual appliance, not a physical device.
- **Direct Connect is supported** — you can use a public VIF or private VIF with a VPC endpoint for the destination service.
- **FSx for Windows** is a common exam pairing: migrate on-prem Windows file servers to AWS using DataSync.
