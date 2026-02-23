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
