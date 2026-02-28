---
tags:
  - aws/ebs
  - storage
  - security
category: Storage
---

# EBS Security & Encryption

## 🔐 EBS Encryption

> [!INFO] Overview
> EBS volumes are stored on physical storage devices within an Availability Zone. By default, encryption is not applied, but it is a best practice for compliance and data security.

*   **At-Rest Encryption**: Provides transparent encryption for EBS volumes and their associated snapshots.
*   **No Extra Cost**: There is no additional charge for using EBS encryption.
*   **Performance**: The OS is completely unaware of the encryption (done at the hardware level), resulting in **zero performance loss**.
*   **Standard**: Uses **AES-256** encryption.

### ⚙️ How it Works (DEK & CMK)
*   **Data Encryption Key (DEK)**: Each volume uses a **unique DEK** to encrypt the data at rest.
*   **Customer Master Key (CMK)**: The DEK is encrypted by a CMK (managed in AWS KMS).
*   **Persistence**: Snapshots and any future volumes created from those snapshots share the same DEK.

### 🛡️ Exam Powerups & Best Practices
*   **Encryption by Default**: You can enable an account-level setting to ensure **all new EBS volumes** are encrypted by default using a default CMK.
*   **Immutability**: You **cannot** change an existing volume from unencrypted to encrypted (or vice-versa) directly. To encrypt an unencrypted volume:
    1.  Create a Snapshot of the volume.
    2.  Copy the Snapshot while enabling encryption.
    3.  Create a new Volume from the encrypted snapshot.
*   **Layered Security**: EBS encryption is separate from OS-level full disk encryption (like BitLocker or dm-crypt). You can use both simultaneously.

---
## 👮 IAM & Access Control
*   **Permissions**: Use IAM policies to control who can create, attach, or delete EBS volumes.
*   **KMS Permissions**: To use encrypted volumes, the IAM principal (or the service role) must have permissions to the relevant **KMS CMK**.
