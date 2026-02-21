---
tags:
  - aws/cloudtrail
  - audit
category: Management & Governance
---

# CloudTrail

> [!INFO] Definition
> AWS CloudTrail logs API calls and user activity across your AWS account. While CloudWatch is for **performance**, CloudTrail is for **auditing** and governance (Who did What, When, and from Where).

## Core Concepts
* **Enabled by Default**: CloudTrail is active the moment you create an AWS account.
* **Event History**: By default, it stores the last **90 days** of management events at no cost.
* **Trails**: To persist logs longer than 90 days or to log Data Events, you must create a "Trail" that sends logs to an **S3 Bucket** or **CloudWatch Logs**.

## Event Types
| Event Type | Description | Exam Note |
| --- | --- | --- |
| **Management Events** | Activities like creating/terminating EC2, creating VPCs (Control Plane). | Logged by default. |
| **Data Events** | High-volume activities like S3 object uploads/downloads or Lambda invocations. | Not logged by default; comes at an extra cost. |

## Trails (Unit of Configuration)
* **Regional Service**: A trail logs events for the region it's created in.
* **One Region vs. All Regions**:
	* **Single Region Trail**: Only logs events for that specific region.
	* **All Regions Trail**: A logical collection that covers all current and future AWS regions automatically.
* **Organizational Trail**: Created in the **Management Account** of an AWS Organization to store logs from all member accounts centrally.

## ⚠️ Key Exam Nuggets
* **Global Service Events**: Services like IAM, STS, and CloudFront log their data to a global endpoint, usually in `us-east-1` (N. Virginia).
* **Latency**: CloudTrail is **not real-time**; there is typically a delay between the API call and the event appearing in the trail.
* **S3 Integration**: To keep logs forever, you must configure a Trail to save them to S3.

---
*Next Topic: [[CloudWatch|Monitoring with CloudWatch]]*
