---
tags:
  - aws/ebs
  - storage
category: Storage
---

# EBS Volume Types

## 💾 General Purpose SSD (GP2/GP3)

### General Purpose SSD (GP3)
> [!INFO] Current standard
> GP3 is the modern successor to GP2. Its biggest advantage is that it **decouples storage from performance**.

*   **Fixed Baseline**: Every GP3 volume gets a baseline of **3,000 IOPS** and **125 MB/s** throughput for free.
*   **Provisioned Performance**: You can independently increase IOPS (to 16,000) or Throughput (to 1,000 MB/s).
*   **Cost**: ~20% cheaper than GP2.
*   **Use Cases**: Virtual desktops, medium-sized databases (MSSQL, Oracle), boot volumes, and interactive apps.

### General Purpose SSD (GP2)
*   **Burst Bucket**: Uses a "Credit" system where performance (IOPS) is tied to volume size (3 IOPS per GiB).
*   **Bursting**: Can burst up to 3,000 IOPS for 30 minutes perfect for OS boots.

---

## 🏎️ Provisioned IOPS SSD (io1/io2)

> [!INFO] Definition
> Designed for storage-intensive applications that require the highest performance, consistent sub-millisecond latency, and high durability.

*   **Performance Control**: You specify exactly how many IOPS you need, independent of storage size.
*   **io2 Block Express**: Supports extreme benchmarks—up to **256,000 IOPS** and **64 TiB** volume sizes.
*   **Durability**: `io2` offers **99.999%** durability (100x better than standard tiers).
*   **Use Cases**: Critical NoSQL/Relational databases (SAP HANA, Oracle, SAS Analytics).

---

## 💿 Hard Disk Drive (HDD) Volumes

> [!ERROR] The HDD "No Boot" Rule
> **HDD-based volumes (st1 and sc1) CANNOT be used as Boot Volumes.**

### Throughput Optimized HDD (st1)
*   **Performance**: High throughput for large, sequential data.
*   **Scaling**: 40 MB/s per TiB (Base) / 250 MB/s per TiB (Burst).
*   **Use Cases**: Big Data, Data Warehouses, Kafka, Log processing.

### Cold HDD (sc1)
*   **Performance**: The lowest cost EBS tier for infrequent access.
*   **Scaling**: 12 MB/s per TiB (Base) / 80 MB/s per TiB (Burst).
*   **Use Cases**: Archives and maximum economy workloads.

---

## 📊 EBS Volume Types Comparison

| Volume Type | Technology | Min Size | Max Size | Max IOPS / Throughput | Best For... |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **GP3** | SSD | 1 GiB | 16 TiB | 16,000 / 1,000 MiB/s | General Purpose (Recommended). |
| **GP2** | SSD | 1 GiB | 16 TiB | 16,000 / 250 MiB/s | Legacy General Purpose. |
| **io2 Block Express** | SSD | 4 GiB | 64 TiB | **256,000** / 4,000 MiB/s | Extreme performance (SAP HANA). |
| **io2** | SSD | 4 GiB | 16 TiB | 64,000 / 1,000 MiB/s | Critical databases & apps. |
| **st1** | HDD | 125 GiB | 16 TiB | 500 / 500 MiB/s | Big Data/Log processing. |
| **sc1** | HDD | 125 GiB | 16 TiB | 250 / 250 MiB/s | Archives/Cold data. |
