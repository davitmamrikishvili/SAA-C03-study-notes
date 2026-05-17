---
tags:
  - aws/elasticache
  - databases
  - caching
category: Databases
---

# ⚡ Amazon ElastiCache

> [!INFO] Definition
> **Amazon ElastiCache** is a fully managed, **in-memory caching service** that delivers sub-millisecond performance. It is used to reduce the load on databases and accelerate read-heavy applications. It supports two open-source engines: **Redis** and **Memcached**.

**Key Characteristics:**
* **In-memory** — data lives in RAM, not on disk. This makes it extremely fast but also non-durable.
* **Managed service** — AWS handles patching, monitoring, failure detection, and recovery.
* **Not persistent by default** — designed for temporary, high-speed data access, not as a primary data store.
* **Requires application code changes** — unlike DAX (DynamoDB Accelerator), ElastiCache is not transparent. Your application must explicitly be written to check the cache before querying the database.

---

## 🏗️ Core Use Cases

### 1. Database Caching (Read-Heavy Workloads)

The most common use case: place ElastiCache in front of a database (RDS, Aurora, DynamoDB) to serve frequently repeated read queries from memory instead of hitting the database each time.

![[Elasticache-1.png]]

**Flow:**
1. Application requests data.
2. **Cache Hit**: Data is in ElastiCache → returned immediately (sub-ms). Database not touched.
3. **Cache Miss**: Data is not in cache → application fetches from the database, writes the result to ElastiCache, and returns it. Future requests for the same data hit the cache.

**Benefit**: Drastically reduces database load (cost) and latency for read-heavy workloads.

### 2. Session State Data (Stateless Architecture)

ElastiCache is an ideal store for **user session data**, enabling your application tier to be fully stateless.

![[Elasticache-2.png]]

**Flow:**
1. User logs in → application writes session token/data to ElastiCache.
2. Any subsequent request — regardless of which EC2 instance handles it — can retrieve the session from the shared ElastiCache cluster.
3. If an EC2 instance fails, the session persists in the cache; another instance picks up seamlessly.

**Benefit**: Enables horizontal scaling and fault tolerance for the application layer.

---

## 🔄 Redis vs. Memcached

These two engines serve different needs. Choosing between them is a common exam scenario.

| Feature                | **Memcached**                       | **Redis**                                                 |
| :--------------------- | :---------------------------------- | :-------------------------------------------------------- |
| **Data Structures**    | Simple strings only                 | Advanced: Lists, Sets, Sorted Sets, Hashes, Bitmaps, etc. |
| **Replication**        | ❌ No replication                    | ✅ Multi-AZ with automatic failover                        |
| **Backups**            | ❌ No backups                        | ✅ Backup & Restore (snapshots to S3)                      |
| **Persistence**        | ❌ None                              | ✅ Optional (AOF / RDB persistence)                        |
| **Transactions**       | ❌ Not supported                     | ✅ Supported (atomic operations)                           |
| **Threading**          | ✅ Multi-threaded (better raw speed) | ❌ Single-threaded (per core)                              |
| **Scaling**            | Horizontal via **Sharding** (nodes) | Horizontal via **Read Replicas** (scales reads)           |
| **Pub/Sub Messaging**  | ❌ Not supported                     | ✅ Supported                                               |
| **Geospatial Support** | ❌ Not supported                     | ✅ Supported                                               |

> [!TIP] Exam Decision Rule
> * Need **HA, replication, backups, advanced data types, or sorted sets** → **Redis**.
> * Need a **simple, fast, horizontally scalable cache** with pure key-value (strings) → **Memcached**.
> * Need to store **session state** or support any **stateless application tier** → **Redis** (due to replication/persistence).

---

## 🔐 Security

* Supports **encryption in transit** (TLS) and **encryption at rest**.
* **Redis AUTH** — optional password/token-based authentication for Redis clusters.
* Deployed **inside a VPC** — not a public service. Access is controlled via **Security Groups**.
* Supports **IAM authentication** (Redis only, via IAM-based auth tokens).

---

## 🎯 Exam PowerUP: When to Choose ElastiCache

> [!IMPORTANT]
> * **Read-heavy workloads** with repeated queries → add ElastiCache as a caching layer.
> * **Session state management** for a stateless web tier → ElastiCache (Redis preferred).
> * **Sub-millisecond latency** requirements → ElastiCache.
> * **Reduces database cost** by offloading repeated reads → ElastiCache.
> * Application code **must be updated** to use ElastiCache — it is never transparent.
> * For DynamoDB-specific caching with **no code changes**, use **DAX** instead.
