---
tags:
  - aws/iac
  - cloudformation
  - devops
category: Management & Governance
---

# 🏗️ AWS CloudFormation

> [!INFO] Definition
> **AWS CloudFormation** is an **Infrastructure as Code (IaC)** service that helps you model, provision, and manage AWS resources safely and predictably. You define your infrastructure in templates, and CloudFormation handles the provisioning process.

---

## 🧱 Key Concepts

* **Templates**: JSON or YAML files detailing the AWS resources you want to create and configure.
* **Stacks**: The runtime instantiation of a template. You create, update, or delete a group of physical resources collectively by managing the stack.
* **StackSets**: Allows you to safely and predictably deploy stacks across **multiple AWS accounts and multiple AWS regions** simultaneously.
* **Drift Detection**: Allows you to detect if physical resources have been manually altered (drifted) outside of the CloudFormation stack management.

---

## ⚙️ Template Structure

A CloudFormation template is broken down into structured sections. The only mandatory section is **Resources**.

| Section                      | Purpose                                                                                                          |
| :--------------------------- | :--------------------------------------------------------------------------------------------------------------- |
| **AWSTemplateFormatVersion** | The template schema version.                                                                                     |
| **Description**              | Text describing the template.                                                                                    |
| **Parameters**               | Custom inputs provided at runtime (e.g., specifying an `InstanceType` when launching the stack).                 |
| **Mappings**                 | Lookup tables (e.g., picking the right AMI IDs depending on the Region).                                         |
| **Conditions**               | Logic that defines whether resources are created or not (e.g., "If environment is Prod, deploy an RDS replica"). |
| **Resources**                | *(Mandatory)* The actual AWS resources to be created.                                                            |
| **Outputs**                  | Values returned after creation (e.g., the Public IP of a web server array) that can be viewed or exported.       |

---

## 🔄 Logical vs. Physical Resources

![[Physical-and-Logical-Resources-1.png]]

* **Logical Resources**: The configuration definitions written in the YAML/JSON template (e.g., `MyWebVPC`).
* **Physical Resources**: The actual, tangible services created in AWS based on the logical definition (e.g., `vpc-01234abcd`).

When a template is executed, CloudFormation translates logical resources into physical resources. If the stack is deleted, the matching physical resources are also destroyed by default. As soon as a logical resource reaches the `CREATE_COMPLETE` state, other resources can reference it to retrieve dynamically generated physical properties (like an assigned IP address).

---

## 📥 Parameters & Pseudo Parameters

### Template Parameters
Parameters allow human users or automated pipelines to provide customized inputs into the template when creating or updating a stack.
* You can configure constraints for each parameter: **Defaults, AllowedValues, MinLength, MaxLength, AllowedPattern, NoEcho** (for passwords/secrets), and **Type**.

![[TemplateParameters-1.png]]

### Pseudo Parameters
Provided natively by AWS. They give you environmental context at runtime without needing manual input.
* Examples: `AWS::AccountId`, `AWS::Region`, `AWS::StackName`, `AWS::NoValue`.

![[PseudoParameters-1.png]]

---

## 🔀 Intrinsic Functions

Intrinsic functions allow your template to dynamically retrieve data, process logic, and format strings at runtime.

* `Ref` / `!Ref`: Retrieves the value of a parameter or the physical ID of a resource.
* `Fn::GetAtt` / `!GetAtt`: Returns the value of a specific attribute from a resource (e.g., the default DNS name of a Load Balancer).
* `Fn::Join` & `Fn::Split`: Manipulate strings.
* `Fn::GetAZs` & `Fn::Select`: Fetch a list of Availability Zones for a region and select a specific index from it.
* `Fn::Sub` / `!Sub`: Substitutes variables within a string at runtime.
* `Fn::Base64`: Frequently used to pass base64-encoded UserData scripts to EC2.
* **Conditions**: `Fn::If`, `Fn::And`, `Fn::Equals`, `Fn::Not`, `Fn::Or`.

---

## 🗺️ Mappings & Variables

Mappings allow information lookup (static mapping keys to values).
* Often used to improve template portability (e.g., mapping specific regions to specific AMI IDs or instance types).
* They use the `!FindInMap` intrinsic function to retrieve data.
* Mappings can have a top-level key and a second-level key.

![[CloudFormationMappings.png]]

---

## 📤 Outputs & Cross-Stack References

Outputs are incredibly useful for returning status information—like the generated URL of a deployed Elastic Load Balancer or the ARN of a specific resource.
* You can **Export** an output with a unique name.
* Other (parent/child) CloudFormation stacks can use the `Fn::ImportValue` function to ingest those exported outputs, effectively supporting **Cross-Stack References**.

![[CloudFormationOutputs.png]]

---

## 🔗 DependsOn & Dependencies

By default, CloudFormation tries to be highly efficient and provisions resources **in parallel**. It natively infers implicit dependencies (e.g., an EC2 instance implicitly depends on the Subnet you placed it in).

However, you sometimes need to enforce an **explicit dependency** sequence.

![[CloudFormationDependsOn.png]]

* **Explicit Dependency**: Adding the `DependsOn` attribute instructs CloudFormation to wait until resource A finishes creation before attempting to create resource B.
* **Common Use Case**: Creating an Elastic IP (EIP) and associating it with an EC2 instance requires an active Internet Gateway (IGW) attached to the VPC. Unless you use `DependsOn` linking the EIP to the IGW attachment, the provisioning might fail due to parallel execution.

---

## ⏳ Bootstrapping: CreationPolicy, WaitCondition & Cfn-Signal

When you provision an EC2 instance, CloudFormation returns `CREATE_COMPLETE` the moment the AWS API provisions the hardware. However, if your instance has a complex `UserData` bootstrapping script (installing packages, pulling code), the instance *isn't fully ready* yet.

To solve this, you use signals.

1. **`cfn-signal`**: A helper script installed on your EC2 instance. It runs at the very end of your user data script to explicitly signal the CloudFormation service that the bootstrapping succeeded (or failed).
2. **Timeout & Failure**: You configure CloudFormation to wait a specific duration (up to 12 hours) and expect a certain number of success signals. If it times out or receives a failure signal, the creation automatically fails and rolls back.

### CreationPolicy
![[CloudFormationCreationPolicy.png]]
Creation Policies are natively attached to resources like **EC2 Instances** or **Auto Scaling Groups**. They automatically instruct CloudFormation to pause the transition to `CREATE_COMPLETE` until it receives the required number of success signals via `cfn-signal`.

### WaitCondition
![[CloudFormationWaitConditions.png]]
A Wait Condition serves the identical purpose but operates as an independent logical resource. You use Wait Conditions when you need to coordinate dependencies between multiple disparate resources (e.g., waiting for an external system or complex set of servers) and where native `CreationPolicy` isn't supported.
