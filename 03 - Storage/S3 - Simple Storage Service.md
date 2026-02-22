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

---
*Next Topic: [[S3 Storage Classes]]*