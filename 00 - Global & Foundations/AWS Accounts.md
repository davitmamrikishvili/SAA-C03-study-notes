---
tags:
  - aws/foundations
  - accounts
category: Cloud Concepts
---

# AWS Accounts

> [!INFO] Definition
> An AWS Account is a **logical container** for resources and identities. It provides a security and billing boundary for your AWS environment.

## The Account Root User
* Created when the AWS account is opened using a unique email address.
* Has **unrestricted access** to all resources and can't be limited by IAM policies (except for Service Control Policies in an Organization).
* **Best Practice**: Enable **MFA** immediately and avoid using it for daily tasks.

## Identity & Access Management (IAM)
* Each account has its own IAM instance.
* IAM is used to create Users, Groups, and Roles with specific (limited) permissions.

## AWS Organizations

> [!INFO] Definition
> AWS Organizations is an account management service that enables you to consolidate multiple AWS accounts into an organization that you create and centrally manage.

### Core Components
* **Management Account**: The master account. Pays for all usage and manages the organization.
* **Member Accounts**: Standard accounts that belong to the organization.
* **Organizational Units (OUs)**: Logical groups of accounts. OUs can be nested (e.g., `Dev` OU inside a `Production` OU).
* **Service Control Policies (SCPs)**: Guardrails that define the maximum permissions for an account or OU.

### Key Benefits & Facts
* **Consolidated Billing**:
	* Single payment method for all accounts.
	* **Volume Discounts**: Combined usage across accounts can qualify for lower pricing tiers (e.g., S3 storage costs).
	* **Reserved Instances (RIs) Sharing**: Unused RIs in one account can be shared with others.
* **Streamlined Management**:
	* Automate account creation (no more credit card manually per account).
	* Less administrative overhead by managing common policies centrally.
* **Security & Governance**:
	* **SCPs**: Even the **Root User** of a member account is restricted by an SCP. But, never the management account.
	* Centrally manage security services like GuardDuty or CloudTrail across all accounts.
	* Tagging policies to enforce cost-tracking consistency.

> [!WARNING] Critical for the Exam
> SCPs define what a principal CANNOT do (Deny) or the maximum it CAN do, but they don't grant permissions on their own—IAM is still required for that

