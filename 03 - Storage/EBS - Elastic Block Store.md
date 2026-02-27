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