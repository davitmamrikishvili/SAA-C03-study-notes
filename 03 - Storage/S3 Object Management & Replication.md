---
tags:
  - aws/s3
  - storage
category: Storage
---

# S3 Object Management & Replication

## ⚙️ Object Management Features

### Object Versioning
* **Default**: Disabled.
* **State**: Once enabled, it cannot be disabled, only **Suspended**.
* **Structure**: Every object gets a `Version ID`. If versioning is disabled, the ID is `null`.
* **Deletion Logic**:
	* Deleting an object creates a **Delete Marker**. Previous versions are retained.
	* Deleting the Delete Marker "restores" the object.
	* To permanently delete, you must specify the `Version ID`.
* **Current Version**: If no version ID is specified, the latest version is retrieved.

### MFA Delete
* Requires Multi-Factor Authentication for two specific actions:
	1.  Changing the versioning state of the bucket.
	2.  Permanently deleting an object version.
* **Benefit**: Protects against accidental or malicious "permanent" deletions.

---

## 🔄 S3 Replication

> [!INFO] Definition
> S3 Replication allows for the automatic, asynchronous copying of objects across buckets.

### Replication Types
* **Cross-Region Replication (CRR)**: Source and destination buckets are in different AWS Regions. Used for global resilience and latency reduction.
* **Same-Region Replication (SRR)**: Both buckets are in the same Region. Used for log aggregation, environment syncing (Prod/Test), or meeting data sovereignty requirements.

### Technical Requirements
* **Versioning**: **MUST** be enabled on both the source and destination buckets.
* **IAM Role**: The source bucket requires an IAM Role with:
    * **Trust Policy**: Allows the S3 service (`s3.amazonaws.com`) to assume the role.
    * **Permissions Policy**: Grants <span style="color:rgb(240, 75, 200)">s3:GetReplicationConfiguration</span>, <span style="color:rgb(240, 75, 200)">s3:ListBucket</span> on the source, and <span style="color:rgb(240, 75, 200)">s3:ReplicateObject</span> on the destination.
* **Encryption**: Data is encrypted in transit via SSL.

### Multi-Account Setup
If the destination bucket is in a **different AWS account**, the destination bucket owner must add a **Bucket Policy** to grant the source account's IAM role permission to replicate objects.

### Configuration Options
* **Scope**: Replicate all objects (default) or a subset filtered by **Prefix** or **Tags**.
* **Storage Class**: You can specify a different storage class for the destination (e.g., move to IA or Glacier immediately).
* **Ownership**: You can choose to have the destination object owned by the source account (default) or the destination account owner.
* **Replication Time Control (S3 RTC)**:
    * **Function**: Replicates **99.99%** of objects within **15 minutes**.
    * **Metrics**: Provides detailed CloudWatch metrics for replication latency and pending operations.
    * **SLA**: Comes with a Service Level Agreement (SLA).

> [!TIP] Exam Nugget: Critical Restrictions
> * **Not Retroactive**: By default, replication only applies to **new objects** created after the rule is enabled. (Existing objects require *S3 Batch Replication*).
> * **One-Way**: Replication is unidirectional (Source $\to$ Destination). It is **not** a bi-directional sync by default.
> * **Encryption Limits**:
>     * **Supports**: Unencrypted, SSE-S3, and SSE-KMS (requires extra key permissions).
>     * **Fails**: Objects encrypted with **SSE-C** cannot be replicated.
> * **No Deletes**: By default, **Delete Markers** and **Permanent Deletions** are **not** replicated. This prevents accidental deletions from propagating across regions.
> * **Won't Replicate**: System events, objects in Glacier/Deep Archive storage classes, or objects that the source bucket owner does not have permissions to read.

### Use Case Summary
| Type    | Scenario                                                                                                                                                                  |
| :------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **SRR** | Log aggregation from multiple buckets into one; Syncing Prod data to Test environments; Compliance to legal rules which don't allow you to store data outside the region. |
| **CRR** | Compliance with geographic backup requirements; Minimizing latency for global users; Regional failover.                                                                   |
