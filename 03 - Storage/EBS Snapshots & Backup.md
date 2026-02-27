---
tags:
  - aws/ebs
  - storage
  - backup
category: Storage
---

# EBS Snapshots & Backup

## 📸 EBS Snapshots

> [!INFO] Definition
> Snapshots are point-in-time, incremental backups of EBS volumes that are stored in **Amazon S3**.

*   **Regional Resiliency**: While EBS volumes are locked to an Availability Zone, snapshots are stored across multiple AZs in a Region (S3). This significantly improves durability.
*   **Incremental Nature**:
    - The first snapshot is a **full copy** of all blocks containing data.
    - Subsequent snapshots only store the **changed blocks**.
    - This saves time and storage costs.
*   **Usage**:
    - **Restore**: You can create a new EBS volume from a snapshot.
    - **Migration**: Snapshots can be copied to other Regions (effectively moving the data across geographical boundaries).
    - **AMI Creation**: AMIs are essentially a combination of a snapshot and configuration data.

---

## ⚡ EBS Snapshots/Volume Performance

### 🛡️ Lazy Loading (Default)
When you create a volume from a snapshot, it is available immediately, but the data is **lazy loaded** from S3.
*   **Behavior**: EBS fetches data as it is requested.
*   **Impact**: Initial reads may have high latency (first-touch penalty).
*   **Requested Blocks**: Any block directly requested by the OS is pulled immediately from S3.

### 🚀 Fast Snapshot Restore (FSR)
*   **Feature**: Provides immediate, full-performance access to data upon volume creation. No lazy loading.
*   **Configuration**: Enabled per-snapshot and **per-AZ**.
*   **Limit**: Up to 50 FSR snapshots per region.
*   **Cost**: Costs significantly more than standard snapshots (billed per hour for the FSR capability).

---

## 💰 Snapshot Consumption & Billing

> [!TIP] How you are billed
> Billing for snapshots is based on the amount of data stored in S3, not the capacity of the source EBS volume.

*   **Data Only**: You are only billed for the blocks used on the disk (e.g., a 100GiB volume with only 5GiB of data results in a 5GiB snapshot bill).
*   **Incremental Efficiency**: Each snapshot only bills for the **unique data blocks** it owns, while referencing blocks owned by previous snapshots in the chain.
*   **Snapshot Deletion**: When you delete a snapshot, only the data unique to that snapshot is removed. If other snapshots still rely on certain blocks, those blocks are automatically preserved and re-assigned to the next snapshot in the chain.
*   **EBS Snapshot Archive**: A low-cost tier for long-term retention of snapshots that are rarely accessed. Retrieval takes 24-72 hours.
*   **Recycle Bin**: Allows you to set a retention period for deleted snapshots to protect against accidental deletion.

---

## 📅 Data Lifecycle Manager (DLM)
A managed service that automates the creation, retention, and deletion of EBS snapshots and EBS-backed AMIs based on **Tags**.
