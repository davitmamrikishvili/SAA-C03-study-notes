---
tags:
  - aws/redshift
  - databases
  - analytics
  - data-warehouse
category: Databases
---

# 🏛️ Amazon Redshift

> [!INFO] Definition
> **Amazon Redshift** is a fully managed, **petabyte-scale data warehouse** service. It is designed for **OLAP (Online Analytical Processing)** — running complex analytical queries against large volumes of aggregated historical data from multiple sources.

**Key Characteristics:**
* **Column-based** storage — optimised for analytical queries that aggregate across many rows but touch only a few columns.
* **Petabyte-scale** — can store and query datasets in the petabyte range.
* **Server-based** (not serverless by default) — you provision a cluster.
* **Runs in a single AZ** — not highly available by design; resilience must be planned explicitly.
* SQL-like interface with **JDBC/ODBC** connector support — compatible with standard BI tools.
* Integrates natively with AWS services: **S3, Glue, QuickSight, Kinesis, DMS**, and more.
* Billed similarly to RDS — pay for the cluster while it is running.

---

## 🏗️ Architecture

![[Redshift-Architecture.png]]

A Redshift cluster consists of one **Leader Node** and one or more **Compute Nodes**.

### Leader Node
* The entry point for all client connections (JDBC/ODBC).
* Receives queries, parses them, and develops an **optimised execution plan**.
* Distributes work to Compute Nodes and aggregates results before returning them to the client.
* **You do not pay for the Leader Node separately.**

### Compute Nodes
* Execute the query fragments assigned by the Leader Node.
* Store the data loaded into Redshift.
* Each Compute Node is partitioned into **Slices**.

### Slices
* Each Compute Node can have **2, 4, 16, or 32 slices** depending on node size.
* Each slice is allocated a portion of the node's memory and disk.
* Slices process their portion of the workload **in parallel**, which is the key to Redshift's query performance.
* The Leader Node manages distributing data and queries across slices.

---

## 🔌 Data Ingestion & Query Extensions

### Redshift Spectrum
* Allows Redshift to **query data in S3 directly**, without loading it into Redshift first.
* Uses independent **Spectrum nodes** (separate from your cluster's Compute Nodes) to execute the S3 portion of a query.
* Ideal for querying cold/archive data in S3 alongside hot data in Redshift in a single query.

### Federated Query
* Allows Redshift to **directly query live data in external sources** such as RDS, Aurora, or S3.
* Useful for joining operational (RDS) and analytical (Redshift) data without a full ETL pipeline.

### COPY Command
* The most efficient way to bulk-load data **from S3 into Redshift**.
* Can also load from DynamoDB, EMR, and remote hosts (SSH).

---

## 🌐 Redshift Enhanced VPC Routing

> [!WARNING] Exam-Critical Feature
> By default, Redshift uses **public routes** when communicating with AWS services (e.g., loading data from S3 via the COPY command). When you enable **Enhanced VPC Routing**, all traffic between Redshift and other services is **routed through your VPC**.
>
> This means you can control it with:
> * **Security Groups**
> * **Network ACLs (NACLs)**
> * **VPC Endpoints** (for private S3 access)
> * **Custom DNS / Route Tables**
>
> **Enable Enhanced VPC Routing whenever network compliance, security, or private routing is required.**

---

## 💾 Resilience & Recovery

Since Redshift runs in a **single AZ**, a full AZ failure will take down the entire cluster. AWS provides several mechanisms to recover:

![[Redshift-DRandResilience.png]]

### In-Cluster Replication
* Data written to a Compute Node is **replicated to 1 additional node** within the cluster.
* Protects against individual node failure within the same AZ.

### Automated Snapshots (to S3)
* Automatic incremental backups occur every **~8 hours** or after **5 GB of data is written**.
* Default retention period: **1 day** (configurable up to **35 days**).
* Stored in S3, which itself replicates data across **3+ AZs** in the same region.
* Manual snapshots are also supported and retained until explicitly deleted.

### Cross-Region Snapshot Copy
* You can configure Redshift to **automatically copy snapshots to another AWS region**.
* In the event of a full regional failure, a new cluster can be **provisioned in the target region** from the snapshot.

> [!TIP] Exam PowerUP: Resilience Strategy
> Redshift is **not HA by design** (single AZ). For exam questions about DR/resilience, the answer is always **Cross-Region Snapshot Copy** — it's the primary mechanism for protecting against both AZ and full regional failure.

---

## 🔄 OLTP vs. OLAP

|                 | **OLTP** (e.g., RDS / Aurora) | **OLAP** (e.g., Redshift)                  |
| :-------------- | :---------------------------- | :----------------------------------------- |
| **Workload**    | Many small, fast transactions | Few large, complex analytical queries      |
| **Storage**     | Row-based                     | Column-based                               |
| **Use Case**    | Day-to-day operational data   | Historical analysis, reporting, dashboards |
| **Data Volume** | GB range                      | TB–PB range                                |

---

## 🎯 Exam PowerUP: When to Choose Redshift

> [!IMPORTANT]
> * **Data warehouse**, **BI reporting**, or **long-term analytics** → **Redshift**.
> * **OLAP / column-based / aggregated historical data** → **Redshift**.
> * Query **S3 data without loading it** via Redshift → **Redshift Spectrum**.
> * Routing Redshift traffic through your VPC (for compliance/security) → **Enhanced VPC Routing**.
> * Data warehouse + visualization → **Redshift + QuickSight**.
> * **Single AZ** — plan for DR explicitly with **Cross-Region Snapshots**.
> * **Ad-hoc, serverless queries on S3** without pre-loading → **Athena** (not Redshift).
