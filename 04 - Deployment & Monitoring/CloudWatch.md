---
tags:
  - aws/monitoring
  - cloudwatch
category: Management & Governance
---

# CloudWatch

> [!INFO] Definition
> Amazon CloudWatch is a monitoring and observability service that provides data and actionable insights for AWS, hybrid, and on-premises applications and infrastructure.

## Three Main Pillars
1. **Metrics**: Variables to measure (CPU Utilization, Disk Read/Write).
2. **Logs**: Centralize logs from various sources (EC2, Lambda, CloudTrail).
3. **Events (EventBridge)**: Trigger actions based on system events or schedules.

## CloudWatch Logs
> [!ABSTRACT] How it Works
> A **Public Service** (usable from AWS or on-premises) that allows you to store, monitor, and access logging data.

### Hierarchy & Structure
* **Logging Source**: EC2, VPC Flow Logs, Lambda, CloudTrail, Route 53, or external apps.
* **Log Event**: The smallest unit; consists of a **Timestamp** and the **Message**.
* **Log Stream**: A sequence of log events from the same source (e.g., a specific instance).
* **Log Group**: A container for multiple log streams of the same type (e.g., all web server logs).
	* **Settings**: Retention, Permissions, and Configuration are defined at the Log Group level.

### Key Features
* **Metric Filters**: Use filters to extract data from logs and turn them into CloudWatch Metrics.
* **Regional Service**: CloudWatch Logs operate within a specific region.

## Alarms (Crucial for Exam)
* **Alarms**: Watch a single metric and perform one or more actions based on the value of the metric relative to a threshold.
	* Example: If CPU > 80% for 5 mins, send an SNS notification.

## Dashboards
* Provide a unified view of your AWS resources across regions.

---
*Next Topic: [[CloudTrail|Auditing with CloudTrail]]*