---
tags:
  - aws/dynamodb
  - databases
  - nosql
category: Databases
---

# ⚡ Amazon DynamoDB

> [!INFO] Definition
> **Amazon DynamoDB** is a fully managed, serverless, NoSQL database service. It is a **public** service (accessible over the internet, or from a VPC via an IGW or **Gateway VPC Endpoint**). It supports both **Key/Value** and **Document** data models.

**Key Characteristics:**
* No self-managed servers or infrastructure — pure DBaaS.
* **Single-digit millisecond** performance at any scale (SSD-backed).
* Highly resilient: data is replicated across multiple AZs by default. Optionally **global** via Global Tables.
* Built-in Backups, Point-in-Time Recovery, and encryption at rest.
* **Event-driven integration** via DynamoDB Streams → Lambda.

---

## 🏗️ Tables, Items & Keys

![[DynamoDBArch-1.png]]

The **Table** is the base entity in DynamoDB — a grouping of **Items** (rows) that share the same primary key structure.

### Primary Keys
When creating a table, you must choose a primary key type. This is immutable after creation.

| Key Type                             | Components                         | When to Use                                                                                                    |
| :----------------------------------- | :--------------------------------- | :------------------------------------------------------------------------------------------------------------- |
| **Simple (Partition Key only)**      | Partition Key (PK)                 | When every item can be uniquely identified by one attribute alone.                                             |
| **Composite (Partition + Sort Key)** | Partition Key (PK) + Sort Key (SK) | When items share a PK value (e.g., all orders for one customer) and the SK provides uniqueness and sort order. |

* **No structural restrictions** beyond the primary key — different items can have completely different attributes (schema-less).
* **Item max size**: **400 KB**.
* **No limit** on the number of items in a table.

---

## ⚙️ Capacity Modes

Think of capacity as **throughput speed**, not raw storage.

### On-Demand
* Designed for **unknown, unpredictable, or spiky workloads**, or when minimizing operational overhead is the top priority.
* You pay **per million Read/Write Request Units** consumed.
* More expensive — can be up to **5× the cost** of Provisioned mode at the same workload.

### Provisioned
* You explicitly configure **RCU** (Read Capacity Units) and **WCU** (Write Capacity Units) per table.
* Every operation consumes at least 1 RCU or WCU.
* Every table also has a **burst pool of 300 seconds** of unused capacity that can absorb temporary spikes.

| Unit      | Definition                                              |
| :-------- | :------------------------------------------------------ |
| **1 WCU** | Write **1 KB** of data per second.                      |
| **1 RCU** | Read **4 KB** of data per second (strongly consistent). |

> [!TIP] Switching Modes
> You can switch between On-Demand and Provisioned capacity modes even after data has been added, though with some restrictions (a cooldown period applies).

---

## 🔢 Capacity Calculations

### WCU Example
> *10 writes/second, 2.5 KB average item size.*
> 1. WCU per item: `⌈2.5 / 1⌉ = 3`
> 2. Total WCUs needed: `3 × 10 = 30 WCUs`

### RCU Example
> *10 reads/second, 2.5 KB average item size.*
> 1. RCU per item: `⌈2.5 / 4⌉ = 1`
> 2. **Strongly Consistent**: `1 × 10 = 10 RCUs`
> 3. **Eventually Consistent**: `10 / 2 = 5 RCUs` (costs half as much)

---

## 🔍 Read Operations: Query vs. Scan

### Query
![[DynamoDB-Query.png]]

* The standard, efficient way to retrieve data from DynamoDB.
* **Must** specify a single **Partition Key (PK)** value.
* Optionally narrow results further with a **Sort Key** value or range.
* Capacity consumed = size of all **returned** items (not scanned). However, filters applied *after* the query still consume capacity for the pre-filtered result set.
* You can request only specific attributes, but you are still charged for the full item size.

### Scan
![[DynamoDB-Scan.png]]

* The **most flexible** but **least efficient** operation.
* Reads every single item in the table. You can filter by any attribute, but capacity is consumed for every item scanned through — not just what is returned.
* **Use only when a Query is architecturally impossible** (e.g., filtering on a non-key attribute across the entire table).

---

## 🔄 Consistency Model

Replication in DynamoDB works by electing one of the AZ storage nodes as a **Leader Node**. All writes are directed to the leader, which then replicates to the other nodes.

![[DynamoDB-Consistency.png]]

| Mode                      | How it Works                                                                   | Cost           | Risk                                                |
| :------------------------ | :----------------------------------------------------------------------------- | :------------- | :-------------------------------------------------- |
| **Eventually Consistent** | Reads from any of the 3 storage nodes (may be stale).                          | ½ RCU per read | Possible stale data immediately after a write.      |
| **Strongly Consistent**   | Reads **always** go to the Leader Node, guaranteeing the most up-to-date data. | 1 RCU per read | No stale data risk; but higher cost, less scalable. |

---

## 💾 Backups

![[DynamoDBArch-2.png]]

### On-Demand Backups
* Similar to manual RDS snapshots — full point-in-time backup retained **until you explicitly delete it**.
* Can be restored to the **same or a different region**.
* Restore options: with or without secondary indexes; optionally change encryption settings.

### Point-in-Time Recovery (PITR)
![[DynamoDBArch-3.png]]

* Must be **explicitly enabled** per table (disabled by default).
* Once enabled, maintains a continuous stream of changes — a **35-day rolling recovery window**.
* Allows restoring to any **1-second-granularity point** within that window, to a **new table**.

---

## 🎯 Exam PowerUP: When to Choose DynamoDB

> [!IMPORTANT]
> * **Key/Value** or **Document** data model → **DynamoDB**.
> * **NoSQL** with serverless, scalable, and low-latency requirements → **DynamoDB**.
> * **Relational data**, JOINs, or complex SQL queries → **NOT DynamoDB** (use RDS/Aurora).
> * **No SQL interface** — access is via AWS Console, CLI, or API (SDK) only.
> * **Billing**: Based on RCU, WCU, Storage, and enabled features (Streams, PITR, etc.).
