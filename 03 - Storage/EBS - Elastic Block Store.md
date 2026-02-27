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
* **Block Storage**: Volumes are presented to the OS as a collection of blocks. They are **mountable** and **bootable**.
* **Network Attached**: Delivered over the network via the AWS backbone, separate from the physical EC2 host.
    * **Independence**: EBS volumes can "follow" an instance if it is moved to a different hardware host.
* **AZ Resiliency**: Data is automatically replicated within a single Availability Zone.
* **Persistence**: Data remains even after the EC2 instance is stopped or terminated.
* **Encryption**: Can be encrypted using **AWS KMS** with zero performance impact.

![[EBS.png]]

---

## 🏗️ EBS Structure & Sub-Topics
To better understand EBS, explore the following split-topic notes:

1. **[[EBS Volume Types]]**: Detailed comparison of GP2, GP3, io2, and HDD tiers (st1/sc1).
2. **[[Instance Store]]**: High-performance local storage and comparisons with EBS.
3. **[[EBS Snapshots & Backup]]**: Backups to S3, Fast Snapshot Restore (FSR), and Lifecycle Manager.

---

## 💰 Pricing & Billing
* **Provisioned Capacity**: Billed per **GB-month** for the amount you provision, not what you use.
* **Snapshots**: Billed based on the amount of **data used** on the volume (stored incrementally in S3).
* **Performance**: GP3 and IO tiers may have separate charges for provisioned **IOPS** and **Throughput**.
