---
tags:
  - aws/rds
  - database
category: Database
---

# 🗄️ Amazon RDS (Relational Database Service)

> [!INFO] Definition
> Amazon RDS is a **Database-as-a-Service (DBaaS)**, though it is more accurately described as a **Database-Server-as-a-Service** product. It provides a fully managed database instance where you can host one or multiple databases.

---

## ⚙️ Core Concepts

* **Managed Service**: AWS handles underlying host maintenance, OS patching, database software patching, backups, and provisioning.
* **Supported Engines**: RDS supports multiple popular database engines:
  * MySQL
  * PostgreSQL
  * MariaDB
  * Oracle
  * Microsoft SQL Server
  * **Amazon Aurora** (AWS's proprietary, high-performance engine)

![[RDSArch-1.png]]

## 🏗️ RDS Database Instances

An RDS **Database Instance** is an isolated database environment running in the cloud.

* **Instance Families**: Similar to EC2, RDS instances come in various families, types, and sizes optimized for different workloads:
  * `db.m5` - General Purpose
  * `db.r5` - Memory Optimized
  * `db.t` families - Burstable Performance
* **Databases per Instance**: A single database instance can contain *multiple* user-created databases (engine permitting).
* **Network Access**: You access the RDS instance using an AWS-provided **Database Hostname (CNAME)**, *not* a static IP address.
* **Storage**: When provisioning, you allocate block storage (**EBS**) for the instance. Both the compute instance and its attached storage exist in the same Availability Zone (AZ). Because of this, a Single-AZ deployment is vulnerable to an AZ-level failure.
* **Security**: Network access to the database instance is tightly controlled via standard VPC **Security Groups**.

### 💰 Billing
You are primarily billed for two things:
1. **Compute**: Hourly cost of the running instance based on its size and family.
2. **Storage**: Provisioned storage capacity (GB per month), regardless of how much data is actually stored.

> [!TIP] Exam PowerUP: High Availability
> A standard RDS deployment is **Single-AZ**. To protect against AZ failures and provide an active/passive automatic failover, you must explicitly configure your RDS instance for **Multi-AZ**.

---

## 🛡️ RDS Multi-AZ (High Availability)

A Standard (Single-AZ) RDS instance and its EBS storage are both located within the same Availability Zone. This makes them vulnerable to **AZ-level outages**. Multi-AZ is the solution used to provide **High Availability (HA)** through an **active/passive** replication model.

### 🔄 How It Works: The Standby Replica
When you enable Multi-AZ, AWS automatically provisions a **Standby Replica** in a **different Availability Zone** within the same region.

* **Synchronous Replication**: When data is written to the primary instance, it is simultaneously replicated to the standby. Both must commit the data to storage before the write is considered successful.
    * **Exam Nugget**: This results in **zero lag** between the primary and standby.
* **Accessibility**: You only ever interact with the **Primary instance**. The Standby replica doesn't have its own accessible endpoint; it is purely a "hot" spare.
* **Automatic Failover**: If AWS detects a failure on the primary, it automatically flips the **DNS CNAME record** (your Database Hostname) to point to the standby's IP.
    * **Detection & Cutover**: Failover typically occurs within **60-120 seconds**.

### 🛠️ What Triggers Failover?
* **Availability Zone Outage.**
* **Primary Instance Failure** (Hardware failure).
* **Manual Failover** (Triggered via "Reboot with failover").
* **Resource Maintenance** (Software patching or instance type changes).

---

> [!IMPORTANT] Exam PowerUP: Multi-AZ Essential Facts
> * **HA, not FT**: Multi-AZ provides High Availability, not "Fault Tolerance" (which would require zero interruption).
> * **Read Scaling? NO**: You **cannot** use a Multi-AZ Standby to scale read performance. It is strictly for failover.
> * **Cost**: You pay for two instances; the Standby is **NOT** part of the AWS Free Tier.
> * **Scope**: Multi-AZ is strictly **Regional** (operates across AZs within one region).
> * **Backups**: Automated backups are taken from the **Standby replica**, ensuring that the backup process does not impact the performance of your Primary instance.


