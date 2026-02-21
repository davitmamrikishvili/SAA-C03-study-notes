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
* **Network Attached (NAS)**: Delivered over the network, making them separate from the EC2 host hardware.
* **Resiliency**: Replicated within an Availability Zone to protect against hardware failure.
* **Persistent**: Data remains even after the EC2 instance is stopped or terminated.

## EBS vs. Instance Store
| Property        | Instance Store                  | EBS                             |
| --------------- | ------------------------------- | ------------------------------- |
| **Type**        | Ephemeral (Temporary)           | Persistent (Permanent)          |
| **Location**    | Physically attached to Host     | Network Attached                |
| **Persistence** | Lost on Host failure/Stop       | Persists past Instance lifetime |
| **Speed**       | Extremely High (Lowest Latency) | High (Varies by type)           |

## Storage Types Comparison
* **Block Storage (EBS)**: Collection of blocks. No structure. **Mountable & Bootable**.
* **File Storage (EFS)**: Presented as a file share with a system structure. **Mountable**, but **not bootable**.
* **Object Storage (S3)**: Flat collection of objects. **Not mountable** and **not bootable**.

## Storage Performance
Performance is measured by how much data can be moved and how fast.

> [!TIP] The Formula
> **IO Size × IOPS = Throughput**

* **IO Size**: The size of each individual block being read or written.
* **IOPS**: Input/Output Operations Per Second.
* **Throughput**: The total volume of data moved per second.

## Key Limitations
* **AZ Affinity**:
	> [!ERROR] Critical Exam Nugget
	> An EBS volume is **locked to a specific Availability Zone**. To move it to another AZ, you must take a **Snapshot** and restore it in the new AZ.
* **Attachment**: Most volumes can only be attached to one instance at a time (unless using **Multi-Attach** on supported Nitro instances).

---
*Next Topic: [[EBS Snapshots & Performance]]*