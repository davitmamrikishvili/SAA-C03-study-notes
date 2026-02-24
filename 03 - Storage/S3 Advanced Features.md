---
tags:
  - aws/s3
  - advanced
category: Storage
---

# S3 Advanced Features

## 🌐 Static Website Hosting

> [!INFO] Definition
> Amazon S3 can host static websites (HTML, CSS, JS, images). It does not support server-side scripting (PHP, Python, etc.).

* **Access**: Allows access via HTTP rather than just AWS APIs.
* **Requirements**: Must define an **Index** document (e.g., `index.html`) and, optionally, an **Error** document.
* **Custom Domains**: If using a custom domain (via Route 53), the **Bucket Name must exactly match the Domain Name**.

### Advanced Use Cases
* **Offloading**: Move static assets (images, videos, JS) from EC2 instances to S3 to reduce compute load and costs.
* **Out-of-band Pages**:
	* **Explanation**: Pages that exist entirely outside of your primary application stack.
	* **Example**: A **Maintenance Page** or **Status Page**. If your VPC, EC2, or RDS fails, the S3-hosted "Down for Maintenance" page remains accessible because it has zero dependency on your main infrastructure.

---

## 🔗 S3 Presigned URL

> [!INFO] Definition
> A Presigned URL gives you temporary access to an object without requiring the requester to have AWS credentials or IAM permissions.

### How it Works
1.  **Generation**: An IAM identity (User or Role) with the necessary permissions "signs" a URL using their credentials.
2.  **Bearer Token**: The resulting URL includes the signature as a query parameter. Anyone who has the URL can use it to perform the specified action (e.g., `GET` or `PUT`).
3.  **Expiration**: The URL is valid only for a specific duration defined at creation.

### Benefits & Use Cases
* **Direct Uploads (PUT)**: Allow users to upload large files (like profile pictures) directly to S3 from their browser, bypassing your application server and saving compute resources.
* **Private Sharing (GET)**: Share a private object (like a digital download link after a purchase) for a limited time without making the bucket or object public.

> [!TIP] Exam Nugget: Security & Limitations
> * **Signer's Permissions**: The URL **does not** "bake in" permissions. S3 checks the signer's current permissions at the *time of access*. If the signer's access is revoked, the URL stops working immediately.
> * **Temporary Credentials (Roles)**: If you generate a URL using an IAM Role, the URL will expire when the role's temporary credentials expire, even if the URL's `ExpiresIn` time is set to be longer.
> * **No Access Case**: You can technically generate a URL for an object you don't have access to, but the URL will simply return `403 Forbidden` when used.

---

## 🔍 S3 Select and Glacier Select

> [!INFO] Definition
> **S3 Select** and **Glacier Select** allow applications to retrieve only a subset of data from an object using simple SQL-like expressions.

*   **The Problem**: Downloading a massive object (up to 5 TB) just to filter a few rows on the client-side is time-consuming, expensive, and wastes bandwidth.
*   **The Solution**: Data filtering happens **at the S3 Service layer**. The client only receives the pre-filtered data.
*   **Performance**: Can provide a significant boost in query performance and cost reduction (up to 400% faster).
*   **Supported Formats**: CSV, JSON, Parquet.
*   **Compression Support**: Works with GZIP or **BZIP2** (for CSV and JSON) and Columnar compression for Parquet.

---

## 🔔 S3 Events

> [!INFO] Definition
> Notifications generated when specific actions occur within an S3 bucket.

*   **Destinations**:
    *   **SNS Topics**: For fan-out to multiple subscribers.
    *   **SQS Queues**: For reliable, decoupled processing.
    *   **Lambda Functions**: For automated serverless workflows.
*   **Requirements**: You must add **Resource Policies** on the destination (SNS/SQS/Lambda) to allow the `s3.amazonaws.com` service principal to publish or invoke.
*   **Format**: The event data is delivered as a **JSON** structure.

### Event Type Definitions

| Event Category     | Action                              | Exam-Relevant Definition                                            |
| :----------------- | :---------------------------------- | :------------------------------------------------------------------ |
| **Object Created** | `Put` / `Post`                      | A new object was uploaded via a simple API call or HTTP form.       |
|                    | `Copy`                              | A new object created by copying an existing source.                 |
|                    | `CompleteMultipartUpload`           | All parts of a large object upload have been assembled.             |
| **Object Deleted** | `Delete`                            | An object has been permanently removed (non-versioned).             |
|                    | `DeleteMarkerCreated`               | In a versioned bucket, a placeholder was added indicating deletion. |
| **Object Restore** | `Post (Initiated)`                  | A request to restore an object from Glacier/Archive has started.    |
|                    | `Completed`                         | The archived object is now temporarily available for access.        |
| **Replication**    | `OperationFailedReplication`        | S3 failed to replicate the object to the destination bucket.        |
|                    | `OperationMissedReplication`        |                                                                     |
|                    | `OperationReplicatedAfterThreshold` |                                                                     |
|                    | `OperationNotTracked`               |                                                                     |

> [!TIP] Exam Nugget: EventBridge
> Use **Amazon EventBridge** as a gateway for S3 events if you need to route to more than 18+ AWS targets, need advanced filtering, or want to bypass the 100-notification-rule limit per bucket.

---

## 📝 S3 Access Logs

> [!INFO] Definition
> Detailed records for every request made to an S3 bucket. Essential for security audits and access analysis.

*   **Architecture**:
    *   **Source Bucket**: The bucket being monitored.
    *   **Target Bucket**: Where logs are delivered.
*   **Mechanism**: Managed by the **S3 Log Delivery Group**.
*   **Best Efforts**: Delivery is "best effort," usually appearing in the target bucket within a few hours. It is **not** guaranteed to be 100% complete or immediate.
*   **Log Structure**: Newline-delimited files containing space-delimited attributes (Requester, Time, Action, Response Code, etc.).

> [!WARNING] The ACL Requirement (Critical)
> To receive logs, you **MUST** manually update the **ACL** on the **Target Bucket** to grant the `S3 Log Delivery Group` **WRITE** and **READ_ACP** (Access Control Policy) permissions. Without this, logs will fail to deliver.

*   **Organization**: It is best practice to use a **Prefix** (e.g., `logs/mysourcebucket/`) in the target bucket to organize logs from multiple sources.
*   **Cost Management**: Manage log growth using **S3 Lifecycle Policies** to transition logs to Glacier or delete them after a retention period.
