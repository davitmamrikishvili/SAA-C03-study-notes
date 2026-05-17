---
tags:
  - aws/athena
  - databases
  - analytics
category: Databases
---

# 🔎 Amazon Athena

> [!INFO] Definition
> **Amazon Athena** is a **serverless, interactive query service** that lets you analyze data directly in **Amazon S3** using standard **SQL**. There are no servers to manage — you pay only for the data scanned by each query.

**Key Characteristics:**
* **Serverless** — no infrastructure to provision or manage.
* Based on **Apache Presto** (distributed SQL engine) under the hood.
* Supports structured, semi-structured, and unstructured data formats: CSV, JSON, ORC, Parquet, Avro.
* Data **never moves** — it stays in S3; Athena queries it in-place.
* Results can be sent to S3, and integrated with other AWS services (QuickSight, Glue, Lambda, etc.).

---

## 🧠 Schema-on-Read

Athena uses a **schema-on-read** model — the critical concept that separates it from traditional databases.

![[AmazonAthena.png]]

* **Traditional databases** enforce a schema *on write* — data must conform to a pre-defined structure before it is stored.
* **Athena** applies the schema *at query time*. Raw data sits in S3 in its original form, unchanged.
* You define a schema (table definition) in advance. When you run a query, Athena projects that schema onto the data as it's being read — transforming it into a table-like structure on the fly.
* **Original data is never modified.** This allows multiple schemas to be applied to the same underlying dataset for different purposes.

> [!TIP] Key Exam Concept
> The phrase **"schema-on-read"** is Athena's defining characteristic. If a scenario mentions querying raw, unstructured, or semi-structured data in S3 without loading it anywhere, Athena is almost certainly the answer.

---

## ⚙️ How It Works

A typical Athena workflow:

1. **Store data in S3** — in any supported format (CSV, JSON, Parquet, ORC, Avro, etc.).
2. **Define a table in the AWS Glue Data Catalog** (or Athena's built-in catalog) — specifying the schema, S3 location, and format.
3. **Run SQL queries** via the Athena console, CLI, SDK, or JDBC/ODBC drivers.
4. **Results** are written to a designated S3 output location and can be viewed immediately.

> [!NOTE] AWS Glue Integration
> Athena uses the **AWS Glue Data Catalog** as its central metadata store. Glue Crawlers can automatically discover and catalog data in S3, creating the table definitions that Athena can then query — reducing manual schema management.

---

## 💰 Pricing Model

* **Pay per query**: Charged based on the **amount of data scanned** ($5 per TB scanned, approximately).
* **No charge** when no queries are running.
* **Cost optimization**: Using **columnar formats** (Parquet, ORC) and **partitioning** your data dramatically reduces the amount of data scanned — and therefore your cost.

| Optimization         | Effect                                                      |
| :------------------- | :---------------------------------------------------------- |
| **Columnar formats** | Only columns referenced by the query are scanned.           |
| **Partitioning**     | Only the relevant data partitions (e.g., by date) are read. |
| **Compression**      | Less data to scan = lower cost and faster queries.          |

---

## 🗂️ Supported Data Sources

While S3 is the primary data source, Athena can also query data from:

* **AWS Glue Data Catalog** (metadata layer over S3)
* **Amazon RDS / Aurora** (via Athena Federated Query)
* **Amazon Redshift** (via Federated Query)
* **DynamoDB, DocumentDB, CloudWatch Logs, and more** — through pre-built Lambda data source connectors.

---

## 🎯 Exam PowerUP: When to Choose Athena

> [!IMPORTANT]
> * **Ad-hoc, interactive querying** of data in S3 → **Athena**.
> * **No ETL, no loading**, query data in its raw format → **Athena** (schema-on-read).
> * **Serverless analytics** — no infrastructure to manage → **Athena**.
> * Need to query **logs** (CloudTrail, ALB access logs, VPC Flow Logs) stored in S3 → **Athena**.
> * Need to run **complex SQL analytics on a pre-loaded data warehouse** → **Redshift** (not Athena).
> * Need a **persistent BI dashboard** → **QuickSight** (can use Athena as a source).
