---
tags:
  - aws/s3
  - storage
category: Storage
---

# S3 - Simple Storage Service (Overview)

> [!INFO] Definition
> Amazon S3 is an **object storage** service that offers industry-leading scalability, data availability, security, and performance.

## 📌 Core Concepts
* **Object-Level Storage**: Data is stored as objects (Key, Value, Version ID, Metadata) in **Buckets**.
* **Global Namespace**: Bucket names must be **globally unique** across all of AWS.
* **Regional Data**: While names are global, data is physically stored within a specific **Region**.
* **Scalability**: Unlimited total storage; single objects can be up to **5 TB**.

### Bucket Naming & Limits
* **Naming**: 3-63 characters, lowercase letters, numbers, and hyphens only. Must start/end with a letter or number.
* **Soft Limit**: 100 buckets per account (requestable up to 1000).

### Public vs Private
* **Private by Default**: All newly created buckets/objects are private.
* **Public Service**: S3 is a public-facing service accessible via public HTTPS endpoints (unless using VPC Endpoints).

---

## 🚀 Performance & Uploads

### Upload Strategies
| Method | Recommendation | Details |
| --- | --- | --- |
| **Single PUT** | < 100 MB | Single blob upload (<span style="color:rgb(240, 75, 200)">s3:PutObject</span>). |
| **Multipart Upload** | > 100 MB | Splits data into parts. Parallelizable and resumable. **Mandatory** for files > 5 GB. |

### S3 Transfer Acceleration
* Uses AWS's global network of **Edge Locations** to route data over the fast AWS backbone instead of the public internet.
* *Restriction*: Bucket name cannot contain periods (`.`).

---

## 💰 Pricing Model
1. **Storage**: GB per month.
2. **Requests**: Per 1,000 operations (PUT, GET, LIST).
3. **Data Transfer**:
	* **IN**: Free.
	* **OUT**: Charged per GB (Free to CloudFront or within the same Region).
4. **Management**: Fees for Intelligent-Tiering monitoring, Storage Lens, etc.

---

## 📂 S3 Deep Dive Topics
Detailed notes on S3 features have been split into the following sections:

1.  **[[S3 Security & Encryption]]**: Policies, Block Public Access, SSE-S3, SSE-KMS, SSE-C.
2.  **[[S3 Storage Classes & Lifecycle]]**: Performance Tiers (Standard, IA, Glacier) and Automation.
3.  **[[S3 Object Management & Replication]]**: Versioning, MFA Delete, CRR, and SRR.
4.  **[[S3 Advanced Features]]**: Static Website Hosting and Presigned URLs.

---
*Next Topic: [[S3 Object Lock & Glacier Vault Lock]]*