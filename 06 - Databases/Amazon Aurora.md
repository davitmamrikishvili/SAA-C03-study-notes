---
tags:
  - aws/aurora
  - database
category: Database
---

# ☄️ Amazon Aurora

> [!INFO] Definition
> Amazon Aurora is a fully managed, MySQL and PostgreSQL-compatible relational database engine. It is designed to combine the speed and availability of high-end commercial databases with the simplicity and cost-effectiveness of open-source databases.

---

## 🏗️ Aurora Architecture

Aurora's architecture is fundamentally different from a standard RDS engine. It is built on a "Cloud Native" foundation.

* **The Cluster**: The base entity in Aurora. A cluster consists of **one primary instance** (reads/writes) and **zero or more replicas** (reads only).
* **Shared Cluster Volume**: Aurora does not use local storage on individual instances. Instead, it uses a **Shared Cluster Volume**.
	* This is a virtual storage layer that is shared and accessible by all instances in the cluster.
	* It spans multiple Availability Zones (AZs) and automatically scales up to 128 TiB.
	* Benefits include faster provisioning, improved availability, and higher performance.

![[AuroraArch-1.png]]

### 🔄 Replication & Resilience
* **Storage-Level Replication**: Replication happens at the storage level, not the instance level.
* **Self-Healing**: Aurora automatically detects failures in the disk segments that make up the cluster volume and repairs them immediately.
* **High Durability**: Data is replicated **6 times across 3 AZs** (2 copies per AZ). Aurora can handle the loss of an entire AZ without any data loss.
* **Failover Replicas**: You can have up to **15 Aurora Replicas**, any of which can be a failover target.

---

## 🔗 Endpoints

Unlike RDS, Aurora clusters provide multiple endpoints to manage connections:

1. **Cluster Endpoint**: Points to the current **Primary instance**. Use this for all Read/Write operations.
2. **Reader Endpoint**: Load balances connections across all available **Read Replicas**. If no replicas exist, it points to the Primary.
3. **Custom Endpoints**: Allow you to group specific instances (e.g., a group of high-performance instances for heavy reporting queries).
4. **Instance Endpoints**: Every instance in the cluster has its own unique endpoint for direct access.

![[AuroraArch-2.png]]

---

## 💰 Costs & Considerations
* **No Free Tier**: Aurora does not have a free-tier option.
* **Compute**: Charged hourly (per second, 10-minute minimum). Aurora does not support small `db.t2.micro` instances.
* **Storage**: Billed based on GB-months consumed and I/O costs per request.
* **Backup Included**: Backups up to 100% of your database size are included in the price.
* **Value**: While more expensive than a single-AZ micro RDS, Aurora offers superior value and performance for production-grade workloads.

---

## 🕒 Advanced Features: Restore, Clone & Backtrack

* **Automated Backups**: Works similarly to RDS, but snapshots are taken against the shared volume.
* **Backtrack**: Allows you to "rewind" the database in-place to a previous point in time without performing a full restore. This is a per-cluster setting.
* **Fast Database Cloning**: Creates a new database cluster from an existing one almost instantly using **Copy-on-Write** (CoW) technology. The clone only takes up extra storage space for the data that has changed from the source.
* **Restores**: Like RDS, a restore always creates a **brand-new cluster**.

> [!TIP] Exam PowerUP: Aurora HA/DR
> * Aurora offers **lower RTO** (failover is faster than the 60-120s of RDS) and **lower RPO** because of the shared storage.
> * If the primary fails, Aurora promotes the replica with the highest priority first. If no priorities are set, it picks based on size, and then arbitrarily.

---

## ☁️ Aurora Serverless

Aurora Serverless is an on-demand, auto-scaling configuration for Aurora. Instead of managing database instances manually, you specify the capacity range, and the database scales based on application demand.

### 📊 Capacity & Scaling: ACUs
* **ACU (Aurora Capacity Unit)**: The unit of measure for compute (CPU) and memory (RAM). Each ACU represents a specific combination of these resources.
* **Scaling Range**: You define a **Minimum** and **Maximum** ACU capacity.
* **Dynamic Scaling**: Aurora automatically scales up, down, and **to zero** (pausing) when there is no activity.
* **Billing**: You are billed per-second for the ACUs used while the database is active. When paused, you only pay for the **Shared Cluster Volume** storage.

### 🏗️ Serverless Architecture
Aurora Serverless utilizes a specialized architecture to enable rapid scaling without connection drops:

1. **Shared Proxy Fleet**: A fleet of proxies managed by AWS that sit between your application and the database. Your application connects to the proxy fleet endpoint.
2. **Resource Pool**: AWS maintains a large, warm pool of ACUs across multiple customers.
3. **The "Warm" Swap**: When scaling is needed, the proxy fleet transparently brokers connections to new ACUs from the warm pool. This prevents the connection resets typical of standard instance resizing.

![[AuroraServerless-1.png]]

### 🎯 Use Cases
Aurora Serverless is ideal for workloads that are **not constant**:
* **Variable/Unpredictable Workloads**: Applications with sudden spikes or long periods of inactivity.
* **Development & Test**: Systems that are only used during business hours.
* **Infrequently Used Apps**: Internal tools or low-traffic sites.
* **New Applications**: When the required capacity is unknown.
* **Multi-tenant Apps**: Consolidating multiple small databases into a single serverless cluster.

> [!IMPORTANT] Exam PowerUP: Serverless vs. Provisioned
> * **Scaling to Zero**: Only Aurora Serverless can pause compute entirely (costing $0 for compute).
> * **Resilience**: Serverless provides the same **6-way replication** across 3 AZs as Provisioned Aurora.
> * **Connectivity**: Requests *must* go through the **Proxy Fleet**, which ensures connections remain intact during scaling events.

## 🌍 Aurora Global Database

Aurora Global Database allows a single Aurora database to span multiple AWS regions. It is designed for globally distributed applications with low-latency read requirements and robust Disaster Recovery (DR).

### 🔄 Global Replication
* **Mechanism**: Replication happens at the **storage level** using a dedicated infrastructure, not the compute instances.
* **Performance**: Cross-region replication typically completes in **under 1 second**, having zero impact on the primary region's database performance.
* **Scale**: You can have **1 Primary Region** and up to **5 Secondary Regions**.
* **Regional Capacity**: Each secondary region can host up to **16 Read Replicas**.

![[AuroraGlobalDatabase.png]]

### 🎯 Use Cases
* **Disaster Recovery (DR)**: A secondary region can be promoted to full R/W capabilities in less than a minute in the event of a regional outage.
* **Global Read Scaling**: Users in secondary regions can read data from local replicas with sub-millisecond latency.

---

## ⚔️ Aurora Multi-Master

In a standard Aurora cluster (Single-Master), only one instance is R/W. In **Multi-Master mode**, every instance in the cluster is a **Writer node** capable of performing both Read and Write operations.

### 🏗️ Architecture & Write Quorum
* **Shared Volume**: All nodes share the same underlying cluster storage.
* **Write Quorum**: When a node receives a write, it proposes the change to the storage nodes. A **Quorum** (majority) of storage nodes must acknowledge the write for it to be committed.
* **Cache Replication**: Once committed to storage, changes are replicated to every other node in the cluster. This allows all writer nodes to update their **in-memory caches** with the most recent data.
* **No Cluster Endpoint**: Unlike Single-Master, there is no single "Cluster R/W Endpoint." Your application must be configured to connect to specific instance endpoints or handle load balancing manually.

![[AuroraMultiMaster-1.png]]

### ⚖️ Single-Master (HA) vs. Multi-Master (FT)
**Single-Master (High Availability)**: If the primary fails, there is a brief interruption while a replica is promoted (60-120s). This is **HA**, but not Fault Tolerant.

![[AuroraMultiMaster-2.png]]

**Multi-Master (Fault Tolerance)**: If a writer fails, the application can immediately shift its traffic to another active writer node with **zero downtime**. This provides the foundation for **Fault Tolerance (FT)**.

![[AuroraMultiMaster-3.png]]

> [!IMPORTANT] Exam PowerUP: Aurora Global & Multi-Master
> * **Global RPO**: Because Global Database replication is asynchronous, the RPO is typically around **1 second**.
> * **Multi-Master Scaling**: Multi-Master is about **availability and continuous uptime**, not necessarily about scaling write performance to infinite levels (due to the overhead of the quorum/conflict resolution).
> * **Endpoints**: Always remember that Multi-Master apps must be "smart"—they need to handle connection management to multiple writer nodes themselves.