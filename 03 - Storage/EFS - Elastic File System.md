---
tags:
  - aws/efs
  - storage
category: Storage
---

# 📂 EFS - Elastic File System

> [!INFO] Definition
> **Amazon EFS** is a fully managed, scalable **Network File System (NFS)** for use with AWS Cloud services and on-premises resources. It provides a simple, serverless, set-and-forget file system that can be mounted on multiple Linux EC2 instances simultaneously.

---

## 🏗️ Architecture & Connectivity

* **NFSv4 Protocol**: EFS is based on the NFSv4 protocol.
* **Shared Storage**: A single EFS filesystem can be mounted on thousands of EC2 instances at the same time, making it ideal for shared applications.
* **VPC Integration**: Access is provided via **Mount Targets** created in each Availability Zone (AZ).
    * Mount targets reside in subnets and have private IP addresses from that subnet's range.
    * Instances connect to these mount targets using the private IP or a DNS name.
* **Isolation**: By default, EFS is private to the VPC it is provisioned into.
* **On-Premises Access**: You can mount EFS on-premises using **Direct Connect (DX)** or **AWS VPN**.

![[EFS-1.png]]

---

## ⚙️ Performance & Throughput Modes

EFS offers two modes each for Performance and Throughput to handle different workload characteristics.

### 🚀 Performance Modes
* **General Purpose (Default)**:
    * Best for latency-sensitive applications (web serving, content management).
    * Offers the lowest per-operation latency (single-digit milliseconds).
* **Max I/O**:
    * Best for massive-scale, highly parallel workloads (big data, media processing, genomic analysis).
    * Scales to higher aggregate throughput and IOPS, but with slightly higher per-operation latency.

### 📈 Throughput Modes
* **Bursting (Default)**:
    * Throughput scales linearly as the filesystem grows (1 GiB of data gives 50 KiB/s of baseline throughput, with a burst bucket).
    * Ideal for applications with variable throughput needs.
* **Provisioned**:
    * Allows you to specify the throughput in MiB/s regardless of the amount of data stored.
    * Ideal for small filesystems that require high throughput (e.g., intensive build processes or reporting).

---

## 💰 Storage Classes & Lifecycle Management

EFS optimizes costs using two main storage tiers:

1. **EFS Standard**: For frequently accessed data.
2. **EFS Infrequent Access (IA)**: For files that are accessed less than once a day. It offers significant cost savings (up to 92% cheaper) but has a fee for each read/write operation.
3. **Lifecycle Management**: Automatically transitions files from Standard to IA after a defined period of inactivity (e.g., 30, 60, or 90 days).

---

> [!IMPORTANT] Exam PowerUP: EFS vs. EBS
> * **Operating System**: EFS is **Linux only** (POSIX compliant). It does not support Windows natively (use FSx for Windows instead).
> * **Availability**: EFS is a **Regional** service. Data is stored redundantly across multiple AZs automatically. (EBS is localized to a single AZ).
> * **Scaling**: EFS scales automatically in terms of storage (pay-per-use), whereas EBS requires manual capacity management.
> * **Security**: Uses standard VPC **Security Groups** to control network access to the Mount Targets.
