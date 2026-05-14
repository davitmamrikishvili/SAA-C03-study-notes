---
tags:
  - aws/snow-family
  - storage
  - migration
  - hybrid
category: Hybrid Connectivity
---

# 🚛 AWS Snow Family

> [!INFO] Definition
> The **AWS Snow Family** provides purpose-built physical devices designed to securely move massive amounts of data **IN and OUT** of AWS. They are the solution when moving data over the internet or Direct Connect would take too long or cost too much.

## ⚙️ How It Works
The process is entirely physical:
1. You log a job in the AWS Console.
2. AWS ships a secure, tamper-evident device (or truck!) to your location.
3. You connect it to your local network and copy data onto it (or off of it).
4. You ship it back to AWS, and they ingest the data into your S3 buckets.

> [!TIP] Security
> All data transferred to a Snow device is heavily encrypted using **AWS KMS**.

---

## 🗂️ The Three Tiers (Exam Essentials)

You must know exactly which device to choose based on **data volume** and **compute requirements**.

### 1. AWS Snowball (Standard)
The foundational physical transport suitcase.
* **Capabilities**: **Storage ONLY** — absolutely no compute capabilities.
* **Capacities**: Available in **50 TB** or **80 TB** sizes.
* **Network**: Connects via 1 Gbps or 10 Gbps.
* **Economical Range**: **10 TB to 10 PB** (you can order multiple devices simultaneously to handle Petabyte-scale migrations).
* **Multi-site**: You can order multiple Snowballs to be delivered to entirely different physical premises simultaneously.

### 2. AWS Snowball Edge
A more advanced, powerful iteration of the standard Snowball suitcase.
* **Capabilities**: **Storage AND Compute** (can run EC2 instances and Lambda functions locally).
* **Use Case**: Ideal for remote sites, ships, or edge locations where data needs to be *processed* (e.g., machine learning, format conversion) locally before it is ingested into the device and shipped to AWS.
* **Networking**: Even faster (10 Gbps to 100 Gbps).
* **Flavours**:
    * **Storage Optimized**: ~80 TB usable, 24 vCPU, 32 GiB RAM, 1 TB SSD.
    * **Compute Optimized**: ~100 TB usage, 52 vCPU, 208 GiB RAM, 7.68 GB NVMe.
    * **Compute with GPU**: Same as above, but with an attached GPU for edge ML processing.

### 3. AWS Snowmobile
An actual 45-foot shipping container pulled by a semi-trailer truck. Literally a portable data center.
* **Capabilities**: Massive-scale data transport.
* **Capacity**: Can store up to **100 PB** of data per truck.
* **Economical Range**: Ideal for a single location migrating **10 PB or more**.
* **Exam Warning**: Snowmobile is special-order only. It is **NOT economical** for multi-site deployments (unless *every* site is huge) or for moving less than 10 PB of data. You use multiple Snowballs for that.

---

## ⚖️ Decision Matrix

| Requirement | Solution to Choose |
| :--- | :--- |
| **< 10 PB, multi-site, storage only** | **Snowball** (Standard) |
| **Needs local processing (EC2) before shipping** | **Snowball Edge** |
| **< 10 PB, edge machine learning** | **Snowball Edge with GPU** |
| **> 10 PB, single massive data center** | **Snowmobile** |
