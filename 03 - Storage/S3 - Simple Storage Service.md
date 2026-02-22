---
tags:
  - aws/s3
  - storage
category: Storage
---

# S3 - Simple Storage Service

> [!INFO] Definition
> Amazon S3 is an **object storage** service that offers industry-leading scalability, data availability, security, and performance.

## Core Properties
* **Object Storage**: S3 stores data as objects (key, value, version ID, metadata). It is **not** a block or file store.
* **Global Namespace**: Bucket names must be **globally unique**.
* **Regional Service**: While names are global, data is stored in a specific region.
* **Scalability**: Unlimited data storage. Single object size limit is **5 TB**.

## Bucket Naming Rules
* 3-63 characters long.
* Lowercase letters, numbers, and hyphens only.
* Must start and end with a letter or number.
* Cannot be formatted as an IP address (e.g., 1.1.1.1).

## Limits
* **Soft Limit**: 100 buckets per account.
* **Hard Limit**: 1000 buckets per account (requestable).

## Public vs Private
* By default, all S3 buckets are private.
* S3 is a **Public Service** (accessible via public endpoints).

## Security & Access Control

### Identity vs. Resource Policies
| Policy Type | Attach To | Use Case |
| --- | --- | --- |
| **IAM (Identity)** | User, Group, Role | Manage permissions across many AWS resources from one place. |
| **Bucket (Resource)** | S3 Bucket | Manage permissions for a specific bucket. **Required** for cross-account or anonymous access. |

> [!TIP] Principal
> Resource policies have a `Principal` block (specifying *who* is affected), which Identity policies do not have.

### S3 Bucket Policies
Used to grant/deny access to buckets and objects.
```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "PublicRead",
			"Effect": "Allow",
			"Principal": "*",
			"Action": ["s3:GetObject"],
			"Resource": ["arn:aws:s3:::my-bucket-name/*"]
		}
	]
}
```

### Access Control Lists (ACLs)
* **Status**: Legacy. Use Bucket Policies instead.
* **Limitation**: Inflexible and simple. Hard to manage at scale.

### Block Public Access
* Safeguard that prevents accidental exposure of data to the public.
* Can be applied at the **Account** or **Bucket** level.

## Static Website Hosting
* **Access**: Allows access via HTTP rather than just AWS APIs.
* **Requirements**: Must define an **Index** document (e.g., `index.html`) and an **Error** document.
* **Custom Domains**: If using a custom domain (via Route 53), the **Bucket Name must exactly match the Domain Name**.

### Advanced Use Cases
* **Offloading**: Move static assets (images, videos, JS) from EC2 instances to S3 to reduce compute load and costs.
* **Out-of-band Pages**:
	* **Explanation**: Pages that exist entirely outside of your primary application stack.
	* **Example**: A **Maintenance Page** or **Status Page**. If your VPC, EC2, or RDS fails, the S3-hosted "Down for Maintenance" page remains accessible because it has zero dependency on your main infrastructure.

## Object Management Features

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

## Performance & Optimization

### Upload Strategies
| Method | Recommendation | Details |
| --- | --- | --- |
| **Single PUT** | < 100 MB | Single blob upload (<span style="color:rgb(240, 75, 200)">s3:PutObject</span>). If the stream fails, the upload fails entirely. |
| **Multipart Upload** | > 100 MB | Splits data into parts (max 10,000). Recommended for files > 100MB. Parts can fail independently and be resumed. |

### S3 Transfer Acceleration
* **How it works**: Uses the global network of **Edge Locations**. Data travels over the fast AWS backbone instead of the public internet.
* **Requirements**:
	* Bucket name must be DNS-compliant.
	* **Cannot contain periods (.)** in the bucket name.

## Pricing Model
1. **Storage**: Charged per GB / month based on the storage class.
2. **Requests**: Charged per 1,000 operations (PUT, GET, LIST, etc.).
3. **Data Transfer**:
	* **IN**: Free.
	* **OUT**: Charged per GB (except to CloudFront or within the same region).


## S3 Encryption

> [!IMPORTANT]
> Buckets themselves are not encrypted; **objects** are. Encryption is defined at the object level.

* **Two Main Methods** (Both are forms of encryption at rest):
	* **Client-side encryption**: You encrypt the data locally before sending it to S3.
	* **Server-side encryption**: You send plaintext data, and AWS infrastructure handles the encryption.

### Server-Side Encryption (SSE) Types
* **SSE-C (Customer-provided keys)**:
	* Customer is responsible for managing the encryption keys.
	* S3 manages the encryption/decryption process, then **discards the key**.
* **SSE-S3 (Amazon S3-managed keys)**:
	* Default method using **AES-256**.
	* AWS handles both the encryption/decryption and key management (new key per object).
	* *Constraint*: Not suitable for heavy regulatory environments; doesn't support role separation.
* **SSE-KMS (KMS-managed keys)**:
	* Uses **Customer Master Keys (CMKs)** in AWS KMS.
	* Provides role separation and audit trails (CloudTrail).
	* Supports both **AWS-managed** and **Customer-managed** CMKs.

### Encryption Summary Table
| Method | Key Management | Encryption Processing | Extras |
| :--- | :--- | :--- | :--- |
| **Client-Side** | YOU | YOU | Full control. |
| **SSE-C** | YOU | S3 | S3 discards keys after use. |
| **SSE-S3** | S3 | S3 | Uses **AES-256**. |
| **SSE-KMS** | S3 & KMS | S3 | Rotation Control & Role Separation. |

### Bucket Default Encryption
* Objects are uploaded using the <span style="color:rgb(240, 75, 200)">putObject</span> operation.
* **Header**: Use `x-amz-server-side-encryption` to specify the method (`AES256` for SSE-S3 or `aws:kms` for SSE-KMS).
* **Default Setting**: You can set a default at the bucket level to ensure all objects are encrypted even if the header is missing.

---

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

> [!SUCCESS] Exam Nugget
> Use **S3 Standard** for **frequently accessed** data which is important and non-replaceable.

#### S3 Standard-IA (Infrequent Access)
* **Cost**: Lower storage cost, but adds a **per GB data retrieval fee**.
* **Billing**: Minimum duration of **30 days** and minimum capacity of **128 KB** per object.

> [!SUCCESS] Exam Nugget
> Use **S3 Standard-IA** for **long-lived** data which is important but where access is infrequent.

#### S3 One Zone-IA
* **Risk**: Stored in a **single AZ**. If that AZ is lost, data is lost.

> [!SUCCESS] Exam Nugget
> Use **S3 One Zone-IA** for **long-lived** data which is **non-critical & replaceable** and where access is infrequent.

#### S3 Glacier Instant Retrieval
* **Performance**: Millisecond access.
* **Billing**: Minimum duration of **90 days**.

> [!SUCCESS] Exam Nugget
> Use **S3 Glacier Instant Retrieval** for **long-lived** data accessed roughly once per quarter with **millisecond** access requirements.

#### S3 Glacier Flexible Retrieval
* **Retrieval Process**: Data must be temporarily retrieved to S3 Standard-IA for access.
* **Retrieval Times**: **Expedited** (1-5 min), **Standard** (3-5 hours), **Bulk** (5-12 hours).

> [!SUCCESS] Exam Nugget
> * **Latency**: Minutes or hours.
> * **Public Access**: You **cannot** make objects public (e.g., for static website hosting).
> * **Usage**: Use for **archival data** where frequent or real-time access isn't needed.

#### S3 Glacier Deep Archive
* **Latency**: Hours or days (Standard: 12h, Bulk: 48h).
* **Billing**: Minimum duration of **180 days**.

> [!SUCCESS] Exam Nugget
> * **Latency**: Hours or days.
> * **Usage**: Archival data that rarely (if ever) needs to be accessed. Ideal for **Legal or Regulatory** data storage.

#### S3 Intelligent-Tiering
* **Tiers**: Frequent, Infrequent, Archive Instant, Archive (Optional), Deep Archive (Optional).
* **Logic**: Automatically moves objects based on 30-day access patterns.

> [!SUCCESS] Exam Nugget
> Use **S3 Intelligent-Tiering** for **long-lived** data with **changing, unknown, or unpredictable** access patterns.

---
*Next Topic: [[S3 Lifecycle Policies]]*