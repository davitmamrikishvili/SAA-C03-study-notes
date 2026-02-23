---
tags:
  - aws/s3
  - storage
category: Storage
---

# S3 Storage Classes & Lifecycle

S3 offers a range of storage classes and lifecycle management tools to optimize costs based on data access patterns.

## 📦 S3 Storage Classes

> [!ABSTRACT] Reliability
> All S3 storage classes (except One Zone-IA) replicate data across at least **3 Availability Zones** and provide **11 nines (99.999999999%)** of durability.

### Comparison Table
| Storage Class | Durability | Availability (AZs) | Min Duration | Min size | Retrieval Fee | Latency |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **S3 Standard** | 11 9s | $\ge$ 3 | None | None | None | Milliseconds |
| **S3 Standard-IA** | 11 9s | $\ge$ 3 | 30 Days | 128 KB | Per GB | Milliseconds |
| **S3 One Zone-IA** | 11 9s | **1** | 30 Days | 128 KB | Per GB | Milliseconds |
| **S3 Glacier Instant** | 11 9s | $\ge$ 3 | 90 Days | 128 KB | Per GB | Milliseconds |
| **S3 Glacier Flexible** | 11 9s | $\ge$ 3 | 90 Days | 40 KB | Per GB | Minutes/Hours |
| **S3 Glacier Deep Archive**| 11 9s | $\ge$ 3 | 180 Days | 40 KB | Per GB | Hours/Days |
| **Intelligent-Tiering** | 11 9s | $\ge$ 3 | None* | None | None | Milliseconds |

### Class Details & Exam Nuggets

#### S3 Standard (The Default)
* **Replication**: $\ge$ 3 AZs.
* **Success Code**: <span style="color:rgb(240, 75, 200)">HTTP/1.1 200 OK</span> upon successful storage.
* **Durability**: 11 nines (Math: If you store 10,000,000 objects, you might lose 1 every 10,000 years).

> [!TIP] Exam Nugget
> Use **S3 Standard** for **frequently accessed** data which is important and non-replaceable.

#### S3 Standard-IA (Infrequent Access)
* **Cost**: Lower storage cost, but adds a **per GB data retrieval fee**.
* **Billing**: Minimum duration of **30 days** and minimum capacity of **128 KB** per object.

> [!TIP] Exam Nugget
> Use **S3 Standard-IA** for **long-lived** data which is important but where access is infrequent.

#### S3 One Zone-IA
* **Risk**: Stored in a **single AZ**. If that AZ is lost, data is lost.

> [!TIP] Exam Nugget
> Use **S3 One Zone-IA** for **long-lived** data which is **non-critical & replaceable** and where access is infrequent.

#### S3 Glacier Instant Retrieval
* **Performance**: Millisecond access.
* **Billing**: Minimum duration of **90 days**.

> [!TIP] Exam Nugget
> Use **S3 Glacier Instant Retrieval** for **long-lived** data accessed roughly once per quarter with **millisecond** access requirements.

#### S3 Glacier Flexible Retrieval
* **Retrieval Process**: Data must be temporarily retrieved to S3 Standard-IA for access.
* **Retrieval Times**: **Expedited** (1-5 min), **Standard** (3-5 hours), **Bulk** (5-12 hours).

> [!TIP] Exam Nugget
> * **Latency**: Minutes or hours.
> * **Public Access**: You **cannot** make objects public (e.g., for static website hosting).
> * **Usage**: Use for **archival data** where frequent or real-time access isn't needed.

#### S3 Glacier Deep Archive
* **Latency**: Hours or days (Standard: 12h, Bulk: 48h).
* **Billing**: Minimum duration of **180 days**.

> [!TIP] Exam Nugget
> * **Latency**: Hours or days.
> * **Usage**: Archival data that rarely (if ever) needs to be accessed. Ideal for **Legal or Regulatory** data storage.

#### S3 Intelligent-Tiering
* **Tiers**: Frequent, Infrequent, Archive Instant, Archive (Optional), Deep Archive (Optional).
* **Logic**: Automatically moves objects based on 30-day access patterns.

> [!TIP] Exam Nugget
> Use **S3 Intelligent-Tiering** for **long-lived** data with **changing, unknown, or unpredictable** access patterns.

---

## ⏳ S3 Lifecycle Configuration

> [!INFO] Definition
> Lifecycle configurations are a set of rules used to automate the management of objects over their lifetime. This helps in cost optimization by moving data to cheaper storage classes or deleting it when no longer needed.

### Core Components
* **Scope**: Rules can be applied to:
	* The **entire bucket**.
	* A **subset of objects** filtered by **Prefix** (folder paths) or **Tags**.
* **Action Types**:
	1. **Transition Actions**: Automatically move objects to a different storage class (e.g., Standard $\rightarrow$ Standard-IA).
	2. **Expiration Actions**: Automatically delete objects or object versions after a specified period.

### Storage Class Waterfall (Transitions)
Transitions are generally designed to move data from **frequent/expensive** storage to **infrequent/cheaper** storage. Below is the typical "colder" movement path:

![[S3LifeCycle.png]]

> [!TIP] Exam Nugget
> * **One-Way Street**: Lifecycle transitions generally go in one direction (**Hot $\rightarrow$ Cold**). You cannot move objects from Glacier back to Standard via Lifecycle rules; you must restore them manually.
> * **Minimum Days**: Transitions often require a minimum number of days to pass since creation (e.g., 30 days for Standard-IA).
> * **Cleanup**: Use Expiration actions to automatically remove **Old Versions** (if versioning is on) or **Incomplete Multipart Uploads** to save storage costs.
> * **Small Objects**: Objects smaller than **128 KB** are not transitioned to some classes (like IA) via lifecycle rules because they are still billed at 128 KB.
