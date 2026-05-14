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

---

## 🗂️ Indexes

Indexes create **alternative views** of a table's data, enabling flexible query patterns beyond the base table's primary key — without duplicating the entire table. All indexes are **sparse**: an item is only added to an index if it contains the attribute used as the index key. Items missing that attribute simply won't appear in the index.

### Local Secondary Indexes (LSI)

* **Must be created at table creation time** — cannot be added later. Max **5 LSIs** per table.
* Allows an **alternative Sort Key (SK)** whilst keeping the same Partition Key (PK) as the base table.
* **Shares the RCU and WCU** of the base table (provisioned capacity mode).
* **Can use Strong Consistency** (because it is co-located with the base table data).

![[DynamoDB-Indexes-1.png]]

### Global Secondary Indexes (GSI)

* **Can be created at any time.** Default limit of **20 GSIs** per table.
* Defines an entirely **alternative PK and SK** — allows querying the table as if it had a completely different key structure.
* **Has its own independent RCU and WCU** allocations (separate from the base table).
* **Always Eventually Consistent** — replication from the base table to the GSI is asynchronous.

![[DynamoDB-Indexes-2.png]]

### Projection: Choosing Attributes

Both LSI and GSI let you choose which base table attributes are **projected** (copied) into the index:

| Projection Type | What Gets Copied                                    |
| :-------------- | :-------------------------------------------------- |
| `KEYS_ONLY`     | Only the index keys (PK, SK). Smallest index.       |
| `INCLUDE`       | Keys + a specific set of named attributes.          |
| `ALL`           | Every attribute from the base table. Largest index. |

> [!CAUTION] Exam PowerUP: Index Design
> * Querying on an attribute that is **NOT projected** into the index will force DynamoDB to fetch the full item from the base table — **extremely expensive**.
> * **Default to GSI.** Use LSI only when you specifically require **Strong Consistency** reads on the alternative view.

---

## 🔔 Streams & Triggers

### DynamoDB Streams

A DynamoDB Stream is a **time-ordered, 24-hour rolling log** of every item-level change (Insert, Update, Delete) in a table. Powered by Kinesis Streams under the hood; enabled on a **per-table** basis.

![[DynamoDBStreams-and-Triggers-1.png]]

Each change record can be configured to include different amounts of data, controlled by the **Stream View Type**:

| View Type            | What's Recorded                                         |
| :------------------- | :------------------------------------------------------ |
| `KEYS_ONLY`          | Only the PK (and SK if applicable) of the changed item. |
| `NEW_IMAGE`          | The entire item state **after** the change.             |
| `OLD_IMAGE`          | The entire item state **before** the change.            |
| `NEW_AND_OLD_IMAGES` | Both the before and after state of the item.            |

### Triggers (Streams + Lambda)

![[DynamoDBStreams-and-Triggers-2.png]]

Triggers combine DynamoDB Streams with AWS Lambda to build **event-driven, serverless workflows**:
1. An item in the table changes → a record is added to the Stream.
2. The Stream event invokes a Lambda function, passing the change data as the event payload.
3. Lambda processes the event — no polling or infrastructure needed.

**Common Use Cases:** Reporting & analytics, aggregation, sending notifications/messages downstream.

---

## 🌍 Global Tables

Global Tables enable **multi-master, multi-region replication** — every replica table can accept both reads and writes.

![[DynamoDB-Globaltables-1.png]]

* **Setup**: Create DynamoDB tables in each desired region, then link them. DynamoDB automatically manages the replication topology and treats them as one logical **Global Table**.
* **Conflict Resolution**: Uses **Last Writer Wins** — if the same item is written to two regions simultaneously, the most recent write (by timestamp) wins and is replicated everywhere.
* **Replication Speed**: Generally achieves **sub-second** replication between regions.

> [!WARNING] Consistency Caveat
> **Strongly Consistent reads are ONLY available in the same region as the write.** Any cross-region read is inherently **Eventually Consistent**, as replication is asynchronous.

**Use Cases**: Global high availability, global disaster recovery, and active-active multi-region architectures.

---

## ⚡ DynamoDB Accelerator (DAX)

DAX is a purpose-built, **in-memory write-through cache** for DynamoDB. Unlike generic caching solutions (ElastiCache), DAX is **natively integrated** — your application uses the DynamoDB API unchanged; the DAX SDK handles cache hits/misses transparently.

![[DynamoDB-DAX-1.png]]

### 🏗️ Architecture

![[DynamoDB-DAX-2.png]]

* DAX is a **cluster** deployed **inside a VPC** (not a public service) across multiple AZs.
* Consists of one **Primary Node** (handles reads + writes) and one or more **Replica Nodes** (read-only), similar to ElastiCache. If the primary fails, a new election occurs.
* Accessed via a **cluster endpoint** that load-balances across all nodes.
* Can be **scaled** UP and OUT.

### Two Cache Types

| Cache           | What it stores                                                   | Operations Involved       |
| :-------------- | :--------------------------------------------------------------- | :------------------------ |
| **Item Cache**  | Individual items by their PK (and SK).                           | `GetItem`, `BatchGetItem` |
| **Query Cache** | Results of Query/Scan operations, including the parameters used. | `Query`, `Scan`           |

### Write Behaviour
* **Write-Through**: Writes go to both DynamoDB and the DAX cache simultaneously, keeping the cache fresh.
* **Cache Miss on Read**: On a miss, DAX fetches the data from DynamoDB, writes it to the primary node's item cache, and returns it — subsequent reads are served from cache.

> [!CAUTION] Exam PowerUP: When NOT to Use DAX
> * Your application requires **Strongly Consistent reads** → DAX only provides eventual consistency. Use DynamoDB directly.
> * Your workload is **write-heavy with very few reads** → the cache provides no benefit, just added cost and complexity.
> * DAX is **private** — it lives inside your VPC. Applications must be in the same VPC (or connected to it) to use DAX.
