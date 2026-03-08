---
tags:
  - aws/dms
  - aws/sct
  - migration
category: Database
---

# 🚚 AWS Database Migration Service (DMS)

> [!INFO] Definition
> **AWS DMS** is a managed service that helps you migrate databases to AWS quickly and securely. The source database remains fully operational during the migration, minimizing downtime for applications that rely on the database.

---

## 🏗️ Architecture & Components

* **Replication Instance**: Essentially a managed EC2 instance running migration software. It sits between the source and target, performing the actual data transfer.
* **Endpoints**: Connection information (credentials, host, port) for the source and target databases. **Crucial Rule**: At least one of the endpoints (source or target) **must** be an AWS service (RDS, Aurora, S3, etc.).
* **Replication Tasks**: The specific "jobs" defined on the replication instance.

![[DatabaseMigrationService.png]]

### 🔄 Migration Task Types
1. **Full Load**: A one-off migration of all existing data from the source to the target. Best for small databases or when downtime is acceptable.
2. **Full Load + CDC (Change Data Capture)**: Migrates existing data and then continuously replicates ongoing changes. This allows for **near-zero downtime** migrations.
3. **CDC Only**: Replicates only the data changes. This is useful if you used an alternative method (like a physical backup restore) to transfer the bulk of the data first.

---

## 🛠️ Schema Conversion Tool (SCT)

DMS is excellent at moving *data*, but it does not natively support complex schema conversions (e.g., converting Oracle-specific stored procedures to PostgreSQL).

* **When to use SCT**: Only required for **Heterogeneous Migrations** (migrating between different database engines, like SQL Server to Aurora MySQL).
* **Homogeneous Migrations**: If moving between the *same* engine (e.g., MySQL to Aurora MySQL), SCT is **not** required, as the schemas are already compatible.

---

## 🏔️ DMS & Snowball (Large Scale Migrations)

For multi-terabyte migrations where internet bandwidth is a bottleneck, DMS can integrate with **AWS Snowball**:

1. **Extract (SCT)**: Use the Schema Conversion Tool to extract database data locally into a generic file format.
2. **Transfer (Snowball)**: Load the extracted data onto a Snowball device and ship it to AWS.
3. **Load (S3 to Target)**: AWS loads the Snowball data into an S3 bucket. DMS then picks up the files from S3 and migrates them into the target database.
4. **Sync (CDC)**: While the physical device is in transit, DMS captures ongoing changes (CDC) from the source and applies them to the target via S3 to ensure the database is up-to-date.

> [!NOTE] Relationship: Why SCT for Snowball?
> Even in homogeneous migrations, SCT is required here because it converts the database into a **generic file format** that can be physically loaded onto the Snowball device.

> [!IMPORTANT] Exam PowerUP: DMS & SCT Nuggets
> * **The Homogeneous Rule**: If the engine type doesn't change, you don't need SCT.
> * **Downtime**: Full Load + CDC is the "magic words" for a **No Downtime** migration.
> * **Interoperability**: One side of the migration **must** be in AWS. You cannot use AWS DMS to migrate a local on-premises database to another local on-premises database.
