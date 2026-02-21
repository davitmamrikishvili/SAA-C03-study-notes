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
* (Conceptual) Allows you to manage multiple AWS accounts centrally.
* Provides consolidated billing and centralized security via **Service Control Policies (SCPs)**.