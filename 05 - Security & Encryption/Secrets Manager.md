---
tags:
  - aws/secretsmanager
  - security
category: Security & Encryption
---

# 🔐 AWS Secrets Manager

> [!INFO] Definition
> **AWS Secrets Manager** is a service specifically designed to securely store, retrieve, and manage sensitive credentials (like database passwords, API keys, and tokens) throughout their lifecycle.

---

## ✨ Key Features

* **Purpose-Built**: Unlike SSM Parameter Store (which holds general configuration), Secrets Manager is strictly optimised for sensitive secrets.
* **Automatic Rotation**: Its defining feature. It can automatically rotate secrets on a schedule using **AWS Lambda** functions (e.g., changing a database password every 30 days automatically).
* **Native Integrations**: Integrates directly out-of-the-box with AWS services like **RDS, DocumentDB, and Redshift** for effortless credential management.
* **Access Methods**: Easily retrieved programmatically via the AWS Console, CLI, API, or SDKs.
* **Cross-Region Replication**: Supports multi-region applications and DR by easily replicating secrets across AWS regions.

![[SecretsManager-1.png]]

---

## 🎯 Exam PowerUP: Secrets Manager vs. SSM Parameter Store

> [!IMPORTANT]
> * **Need automatic rotation?** → **Secrets Manager**.
> * **Direct RDS/Aurora credential integration?** → **Secrets Manager**.
> * **Storing basic configuration strings or AMIs?** → **SSM Parameter Store** (cheaper/free).
> * **Basic SecureString storage without rotation?** → **SSM Parameter Store** is usually the correct, cost-effective answer on the exam *unless* automatic rotation is explicitly required.
