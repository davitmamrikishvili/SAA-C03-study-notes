---
tags:
  - aws/s3
  - security
  - encryption
category: Storage
---

# S3 Security & Encryption

S3 provides multiple layers of security to protect data both at rest and in transit.

## 🔐 Access Control

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

---

## 🛡️ S3 Encryption

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
