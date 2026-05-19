---
tags:
  - aws/config
  - compliance
  - governance
category: Security & Encryption
---

# 📋 AWS Config

> [!INFO] Definition
> **AWS Config** is a service that enables you to assess, audit, and evaluate the configurations of your AWS resources. It continuously monitors and records your AWS resource configurations and allows you to automate the evaluation of recorded configurations against desired guidelines.

---

## ✨ Key Features & Characteristics

* **Resource Recording**: Monitors the configuration of every resource in your account. Any time a resource changes, a Configuration Item (CI) is created, detailing what changed, its relationship to other resources, and who made the change.
* **Auditing & Compliance**: Checks if resources comply with predefined organisational standards (e.g., "Are all EBS volumes encrypted?").
* **Storage**: All configuration data and change histories are safely stored in an S3 bucket in a consistent format.
* **No Prevention**: AWS Config **does not prevent** changes from happening (it provides administrative governance, not active protection).
* **Regional Service**: Config operates regionally, but can be configured for cross-region and cross-account aggregation to provide an organisational-wide view.

## 🔗 Rules & Remediation

![[AWSConfig-1.png]]

* **Config Rules**: You can use AWS-managed rules or define custom rules using **AWS Lambda**.
* **Evaluation**: Resources are continuously evaluated against these rules and marked as either **Compliant** or **Non-Compliant**.
* **Event-Driven Actions**: Configuration changes or non-compliance states can trigger events via **EventBridge**, which can then invoke **SNS** (for notifications) or **Lambda** (for automatic remediation).

> [!TIP] Exam PowerUP
> * Tracking configuration history over time → **AWS Config**.
> * Checking compliance against internal standards/rules → **AWS Config**.
> * Auditing *who* made an API call globally → **CloudTrail** (Often compared: CloudTrail tracks *actions*, Config tracks the *resulting state/compliance*).
