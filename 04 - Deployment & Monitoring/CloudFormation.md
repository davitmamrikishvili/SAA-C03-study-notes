---
tags:
  - aws/iac
  - cloudformation
  - devops
category: Management & Governance
---

# CloudFormation

> [!INFO] Definition
> AWS CloudFormation is a service that helps you model and set up your AWS resources so that you can spend less time managing those resources and more time focusing on your applications. It is **Infrastructure as Code (IaC)**.

## Template Structure (YAML/JSON)
The only mandatory section is **Resources**.

| Section | Purpose |
| --- | --- |
| **AWSTemplateFormatVersion** | The date the template was written. |
| **Description** | Text describing the template. |
| **Parameters** | Custom values you provide at runtime (e.g., InstanceType). |
| **Mappings** | Lookup table (e.g., AMI IDs per region). |
| **Resources** | **Mandatory**. The actual AWS resources to be created. |
| **Outputs** | Values returned after creation (e.g., Public IP). |

## Key Concepts
* **Stacks**: A single unit of management; you create, update, or delete a group of resources by managing the stack.
* **StackSets**: Allows you to deploy stacks across multiple accounts and regions.
* **Drift Detection**: Detects if resources have been manually changed outside of CloudFormation.

---
*Next Topic: Intrinsic Functions (!Ref, !GetAtt)*
