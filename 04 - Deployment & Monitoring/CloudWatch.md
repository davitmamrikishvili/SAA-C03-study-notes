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

## Alarms (Crucial for Exam)
* **Alarms**: Watch a single metric and perform one or more actions based on the value of the metric relative to a threshold.
	* Example: If CPU > 80% for 5 mins, send an SNS notification.

## Dashboards
* Provide a unified view of your AWS resources across regions.

---
*Note: CloudWatch is for performance monitoring. CloudTrail is for API auditing.*