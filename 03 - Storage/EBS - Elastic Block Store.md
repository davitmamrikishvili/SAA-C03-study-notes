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
