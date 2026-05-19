---
tags:
  - aws/guardduty
  - security
  - threat-detection
category: Security & Encryption
---

# 🛡️ Amazon GuardDuty

> [!INFO] Definition
> **Amazon GuardDuty** is a continuous security monitoring service that analyses and processes baseline account activity to identify unexpected and unauthorised (potentially malicious) activity within your AWS environment.

---

## ✨ Key Features & Characteristics

* **Continuous Threat Detection**: Uses Machine Learning (ML), anomaly detection, and integrated threat intelligence feeds (lists of known bad IP addresses and domains) to identify active threats.
* **No Agents Required**: It operates entirely at the infrastructure level by natively consuming AWS logs, meaning there is zero performance impact on your workloads.
* **Multi-Account**: Supports centralized management using AWS Organizations (Master and Member accounts).

## ⚙️ Architecture & Data Sources

GuardDuty functions silently in the background by ingesting and analyzing several foundational log streams:

![[AmazonGuardduty-1.png]]

1. **VPC Flow Logs**: Monitors traffic moving through the VPC to detect unusual communication (e.g., instances communicating with known crypto-mining pools or command-and-control servers).
2. **CloudTrail Event Logs**: Detects unusual API activity (e.g., turning off logging, launching unexpected resources in unused regions).
3. **CloudTrail S3 Data Events**: Monitors object-level interactions to detect anomalous data access patterns.
4. **Route 53 DNS Logs**: Detects DNS exfiltration or queries to malicious domains.

## 🚨 Findings & Remediation

When GuardDuty detects a threat, it generates a **Finding** with a specific severity level.
* Findings can be sent to **EventBridge**.
* EventBridge can route the event to **SNS** (for alerting your security team) or **AWS Lambda** (for automated remediation, like isolating a compromised EC2 instance by modifying its Security Group).

> [!TIP] Exam PowerUP
> * **Intelligent threat detection** using ML and foundational logs (VPC Flow Logs, CloudTrail, DNS) → **Amazon GuardDuty**.
> * GuardDuty looks for *active threats and anomalies*. Amazon Macie looks for *sensitive data*. Amazon Inspector looks for *software vulnerabilities*.
