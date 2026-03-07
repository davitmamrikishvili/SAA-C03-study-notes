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

---

## 💾 RDS Backups and Restores

Understanding how RDS protects data is critical for disaster recovery (DR) planning. This involves two key metrics:

* **RPO (Recovery Point Objective)**: The maximum age of files that must be recovered from storage for normal operations to resume. Effectively: **"How much data can we afford to lose?"**
* **RTO (Recovery Time Objective)**: The maximum amount of time allowed to recover from a failure. Effectively: **"How long can we afford to be down?"**

![[RTOvRPO.png]]

### 🧬 Backup Types

RDS backups are stored in **AWS-managed S3 buckets**. These buckets are not visible in your personal S3 console.

#### 1. Automated Backups
* **Mechanics**: Daily full snapshots of the entire instance plus transaction logs captured every 5 minutes.
* **Retention**: Configurable between **0 to 35 days**. Setting it to `0` disables automated backups.
* **Point-in-Time Recovery (PITR)**: Allows you to restore to any point in time during the retention period (down to the second) by replaying transaction logs against the last full snapshot.
* **Deletion**: By default, automated backups are deleted when the RDS instance is deleted.

#### 2. Manual Snapshots
* **Mechanics**: User-initiated full (initially) and incremental (onward) copies of the database volume.
* **Lifecycle**: They do **not expire** and are not automatically deleted. They persist even after the RDS instance is deleted.
* **Performance Impact**: During a snapshot, there is a brief I/O suspension. In a Multi-AZ deployment, this impact is mitigated because the snapshot is taken from the **Standby Replica**.

### 🏥 The Restore Process

Restoring a database in RDS is **not** a "place-in-place" operation.

* **New Instance**: Every restore creates a **brand new RDS instance**.
* **New Endpoint**: The new instance generates a **new DNS CNAME endpoint**. You MUST update your application configuration to point to this new address.
* **RTO Considerations**: Restores take time (provisioning a new instance + data transfer). This is a critical factor when calculating your RTO.

> [!IMPORTANT] Exam PowerUP: Backup & Restore Nuggets
> * **Corruption Protection**: Replication (Multi-AZ) can replicate corruption. Manual Snapshots are the only true protection against accidental data corruption or deletion.
> * **Final Snapshot**: When deleting an RDS instance, AWS always offers to create a "Final Snapshot." Always say yes in a production environment.
> * **Storage Type**: Backups and snapshots are stored on S3 for 11 nines of durability.
> * **Standby Advantage**: Always remember that in Multi-AZ, backups are taken from the standby instance to ensure **zero performance impact** on the primary production workload.

---

## 📚 Read-Replica Architecture

Read replicas allow you to scale the **read performance** of your database by offloading read-only traffic from the primary instance. Unlike standby replicas in Multi-AZ, read replicas are active and accessible.

### 🔄 Replication Mechanics: Asynchronous
Read replicas utilize **Asynchronous Replication**.
* **The Flow**: Data is written and committed to the primary instance's storage first. Once stored, the change is then shipped and applied to the read replica(s).
* **The Lag**: Because replication happens *after* the primary commit, there is a theoretical "Replica Lag." In steady states, this is usually near zero, but it can increase during high write volumes.

### 🌍 Global Reach & Scaling
* **Performance Scaling**: You can have up to **5 direct read replicas** per database instance. Each replica provides an additional endpoint for read-heavy operations.
* **Nested Replicas**: You can create replicas *of* replicas, but be aware that this significantly increases replication lag.
* **Cross-Region Replicas**: You can place replicas in different AWS regions to provide low-latency reads to global users. AWS handles the complex encrypted networking between regions automatically.

### 🚑 Disaster Recovery: Promotion
If the primary database fails (and you aren't using Multi-AZ) or if you want to migrate, you can **Promote** a read replica to become a standalone, primary database.
* **Low RTO**: Promotion is faster than restoring from a snapshot.
* **Irreversible**: Once promoted, the instance becomes a completely independent database and cannot be "demoted" back to a replica.
* **Corruption Risk**: Because replication is asynchronous, a promoted replica might miss the very last transaction that occurred on the primary before a crash. Additionally, read replicas *will* replicate data corruption.

> [!IMPORTANT] Exam PowerUP: Multi-AZ vs. Read Replicas
> This is a foundational exam concept. Know the difference:

| Feature              | Multi-AZ (Standby)         | Read Replicas                   |
| :------------------- | :------------------------- | :------------------------------ |
| **Replication Type** | **Synchronous** (Zero Lag) | **Asynchronous** (Possible Lag) |
| **Primary Goal**     | High Availability (HA)     | Read Scaling / Performance      |
| **Accessibility**    | **NOT** Accessible         | **Accessible** (Read-Only)      |
| **Scope**            | Regional (Across AZs)      | Regional or **Cross-Region**    |
| **Failover**         | **Automatic** DNS Flip     | **Manual** Promotion            |
| **Backups**          | Taken from Standby         | Taken from Primary              |

---

## 🛡️ RDS Security

Security in RDS is implemented across multiple layers, covering both data protection and access control.

### 🔒 Encryption (At Rest & In Transit)

* **In-Transit**: Encrypted using **SSL/TLS**. You can enforce mandatory SSL connections at the parameter group or network level.
* **At-Rest (EBS Encryption)**: Utilizes **AWS KMS**.
    * **Mechanism**: Handled by the underlying Host/EBS volume. RDS uses the CMK (Customer or AWS Managed) to generate data keys for encryption.
    * **Scope**: Storage, system logs, manual snapshots, and read replicas are all encrypted using the same CMK.
    * **Irreversibility**: Once an RDS instance is created with encryption, it **cannot be removed**. To "decrypt" a database, you must export the data and import it into a new, unencrypted instance.

#### 🏛️ Specialty Encryption
* **TDE (Transparent Data Encryption)**: Supported specifically by **MSSQL** and **Oracle**. This is engine-native encryption.
* **CloudHSM Integration**: RDS Oracle supports integration with **AWS CloudHSM**, providing the highest level of cryptographic key control (FIPS 140-2 Level 3).

![[RDSSecurity-1.png]]

### 🔑 Authentication: IAM vs. Local

While standard database logins use internal local DB users, RDS uniquely supports **IAM Database Authentication**.

* **Mechanism**:
    1. Create a local database user account.
    2. Configure it to allow authentication using an AWS Authentication Token.
    3. Attach an IAM Policy to a user or role allowing the <span style="color:rgb(240, 75, 200)">**`generate-db-auth-token`**</span> operation.
* **The Token**: This operation generates a signed, temporary token with a **15-minute validity period**, used in place of a password.
* **Authentication vs. Authorization**: IAM handles the *Authentication* (proving who you are). However, the internal DB engine still handles *Authorization* (what you can do); permissions must be assigned to the local DB user.

![[RDSSecurity-2.png]]

> [!IMPORTANT] Exam PowerUP: RDS Security Nuggets
> * **Encryption Requirement**: You must enable encryption at the time of creation. You cannot encrypt an existing unencrypted RDS instance directly; you must take a snapshot, copy it to an encrypted version, and restore.
> * **IAM Auth Benefit**: It eliminates the need to manage database passwords within your application code (use Instance Profiles instead).
> * **Network Security**: Always remember that Security Groups are the primary tool for controlling network access (ingress/egress) for RDS instances.

## ☄️ Amazon Aurora

Amazon Aurora is AWS's proprietary, cloud-native database engine which offers significant architectural differences and advantages over standard RDS engines.

> [!INFO] Deep Dive
> For detailed architectures, endpoints, and advanced features like Backtrack and Fast Clones, see the dedicated note: [[Amazon Aurora]].