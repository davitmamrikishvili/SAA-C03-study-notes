---
tags:
  - aws/iam
  - security
  - foundations
category: Security, Identity & Compliance
---

# IAM - Identity Access and Management

> [!INFO] Definition
> IAM is a **global service** that allows you to manage access to AWS services and resources securely. It identifies who is making the request (Authentication) and what they are allowed to do (Authorization).

## Core Concepts
* **Global Service**: IAM is globally resilient; it does not depend on a single AWS Region.
* **Cost**: IAM is offered at no additional charge.
* **IAM Identities**:
	* **User**: Long-term credentials for humans or applications. (Max: 5000 per account)
	* **Group**: A collection of users to simplify permission management (e.g., Admins, Developers).
	* **Role**: Temporary identities for AWS services or external users/accounts. Uses **STS** (Security Token Service) for temporary credentials.

## IAM Components
* **Policies**: JSON documents that define permissions.
	* **Managed Policies**: AWS-managed or Customer-managed; can be attached to multiple entities.
	* **Inline Policies**: Embedded directly into a single user, group, or role.
* **Access Keys**:
	* Used for CLI/SDK access (Programmatic access).
	* Consists of an **Access Key ID** and a **Secret Access Key**.
	* Max 2 active keys per user.

## The Account Root User
* Created when the AWS account is opened.
* Has **full, unrestricted access** to all resources in the account.
* **Best Practice**: Use it only to create your first IAM user, then lock it away with MFA.

## Key Limits & Best Practices
* **Principle of Least Privilege**: Grant only the permissions required to perform a task.
* **IAM Groups**: A user can be a member of up to **10 groups**.
* **MFA**: Always enable Multi-Factor Authentication for the root user and privileged IAM users.

---
*Note: Continuing from Cantrill's Course Section: IAM Basics*