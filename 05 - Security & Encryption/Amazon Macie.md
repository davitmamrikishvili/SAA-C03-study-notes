---
tags:
  - aws/macie
  - security
  - data-privacy
category: Security & Encryption
---

# 🕵️ Amazon Macie

> [!INFO] Definition
> **Amazon Macie** is a fully managed data security and data privacy service that uses machine learning and pattern matching to randomly discover and protect sensitive data stored in **Amazon S3**.

---

## ✨ Key Features & Characteristics

* **S3 Exclusivity**: Macie focuses specifically on discovering and protecting data stored inside S3 buckets.
* **Sensitive Data Discovery**: Automatically identifies Personally Identifiable Information (PII), Personal Health Information (PHI), financial data, and credentials.
* **Multi-Account**: Uses a multi-account architecture via AWS Organizations (a central delegated admin account monitors member accounts).

## ⚙️ How It Works (Data Identifiers)

Macie assesses objects against two types of rules:
1. **Managed Data Identifiers**: Built into the product using ML/pattern matching to find standard data (credit cards, passports, AWS Secret Keys).
2. **Custom Data Identifiers**: Proprietary, regex-based identifiers you create (e.g., specific company employee ID formats).

You create **Discovery Jobs** (scheduled or on-demand) which scan chosen S3 buckets using these identifiers.

![[AmazonMacie-1.png]]

## 🔍 Findings & Integrations

If Macie detects issues, it generates **Findings**.

1. **Policy Findings**: Generated when bucket settings are changed insecurely (e.g., turning off encryption or making the bucket public). Examples: `Policy:IAMUser/S3BucketPublic`.
2. **Sensitive Data Findings**: Generated when actual sensitive data is detected inside an object. Examples: `SensitiveData:S3Object/Financial`.

Findings can be viewed in the console, sent to **AWS Security Hub**, or routed to **EventBridge** (which can then trigger Lambda for automatic remediation).

> [!TIP] Exam PowerUP
> If a question mentions discovering **PII**, **PHI**, or **sensitive data in S3 buckets** using Machine Learning → **Amazon Macie**.
