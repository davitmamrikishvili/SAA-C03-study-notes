---
tags:
  - aws/ssm
  - security
  - configuration
category: Security, Identity & Compliance
---

# 📝 AWS SSM Parameter Store

AWS Systems Manager Parameter Store provides secure, hierarchical storage for configuration data management and secrets management.

---

## 🛠️ Core Capabilities
* **Storage Types**:
    * **String**: Plaintext configuration values.
    * **StringList**: A comma-separated list of strings.
    * **SecureString**: Encrypted data using **AWS KMS**.
* **Hierarchies**: Parameters can be organized into hierarchies (e.g., `/prod/db/hostname`). This allows for granular IAM permissions and bulk retrieval by path.
* **Versioning**: Every change to a parameter creates a new version. You can track history and rollback if needed.
* **Service Type**: It is a **Public Service** (accessible via public API endpoints).

---

## 🔐 Security & Encryption
* **KMS Integration**: For `SecureString` parameters, SSM integrates with AWS KMS to encrypt the value at rest.
* **IAM Permissions**: To retrieve a `SecureString`, a user/role needs:
    1. `ssm:GetParameter` (to access the value in SSM).
    2. `kms:Decrypt` (to decrypt the value using the specific KMS key).
* **Privacy**: High-security values like passwords, API keys, and license codes should always be stored as `SecureString`.

---

## 🚀 Advanced Features
* **Parameter Policies**: Allows you to set expiration dates (TTL), notification policies, and "No-Change" windows for parameters.
* **EventBridge Integration**: Changes to parameters can trigger events in **Amazon EventBridge**, allowing for automated responses to configuration drift.
* **Public Parameters**: AWS provides public parameters (e.g., Ami IDs for latest Amazon Linux) that you can reference in your templates.

> [!TIP] Exam PowerUP: Parameter Store vs. Secrets Manager
> * Use **Parameter Store** for general configuration and basic secrets (it's often free/cheaper).
> * Use **Secrets Manager** if you need **Automatic Rotation** or integration with RDS/Redshift passwords.
