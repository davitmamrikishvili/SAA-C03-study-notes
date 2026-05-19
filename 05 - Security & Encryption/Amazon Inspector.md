---
tags:
  - aws/inspector
  - security
  - vulnerabilities
category: Security & Encryption
---

# 🔎 Amazon Inspector

> [!INFO] Definition
> **Amazon Inspector** is an automated vulnerability management service that continually scans AWS workloads for software vulnerabilities and unintended network exposure.

---

## ✨ Key Features & Characteristics

* **Primary Targets**: Designed specifically to inspect **EC2 instances**, container images in ECR, and Lambda functions.
* **Vulnerabilities**: Checks for deviations from best practices, unintended network exposure, and known software Common Vulnerabilities and Exposures (CVEs).
* **Assessments**: Runs continuously (or on a schedule: 15 mins, 1 hr, 1 day) and provides a highly contextualized risk score for each finding.

## ⚙️ Assessment Types

Inspector primarily works through two layers of assessment on EC2 instances:

1. **Network Assessment (Agentless)**: Checks how an instance or group of instances is exposed to the public internet end-to-end (checks EC2, ELB, SGs, NACLs, Route Tables, IGWs).
   * Returns findings such as `RecognizedPortWithListener`.
2. **Host Assessment (Agent Required)**: Requires the SSM Agent (or dedicated Inspector agent). Looks strictly at OS-level vulnerabilities, missing patches, and insecure application configurations inside the instance itself.

## 📋 Rules Packages

Inspector evaluates workloads based on built-in rules packages, which heavily feature:
* **Common Vulnerabilities and Exposures (CVE)**.
* **Center for Internet Security (CIS) Benchmarks**.
* **Security best practices for Amazon Inspector**.

> [!TIP] Exam PowerUP
> * Scanning **EC2 instances** for **OS vulnerabilities (CVEs)** or unintended network exposure → **Amazon Inspector**.
> * Protecting your network from DDoS at the perimeter → **AWS Shield**.
> * Protecting your web applications from SQL injection → **AWS WAF**.
