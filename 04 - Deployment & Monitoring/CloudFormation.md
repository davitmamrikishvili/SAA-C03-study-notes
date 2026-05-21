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

## � Conditions

Conditions allow your stack to dynamically react and change which infrastructure is deployed (or how it is configured) based on specific situational criteria.

![[CloudFormationConditions.png]]

* **Evaluation**: Conditions always evaluate to `True` or `False`.
* **Execution Order**: They are processed *before* logical resources are instantiated.
* **Intrinsic Functions**: Uses logical intrinsic functions such as `Fn::And`, `Fn::Equals`, `Fn::If`, `Fn::Not`, and `Fn::Or`.
* **Resource Attachment**: Any logical resource can have a condition associated with it. If the condition evaluates to true, the resource is created; if false, the resource is ignored block-wide.
*(e.g., Only creating a read-replica database if the "EnvironmentType" parameter equals "Production")*.

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

---

## 🪆 Nested Stacks

Nested stacks allow you to break down large, monolithic CloudFormation templates into smaller, reusable templates.

![[CloudFormationNestedStacks-1.png]]

![[CloudFormationNestedStacks-2.png]]

* **Root Stack & Parent Stacks**: You deploy a central "Root Stack," which natively references and creates other underlying "Nested Stacks" using the `AWS::CloudFormation::Stack` logical resource type.
* **Overcoming Limits**: Standard CloudFormation stacks have a limit of 500 resources. Nested stacks easily bypass this limitation by chaining stacks together.
* **Modular Code Reuse**: Instead of copying and pasting the exact same VPC configuration into 10 different templates, you can create one generic VPC template and *nest* it inside your other templates as a reused module.
* **Lifecycle Linking**: Nested Stacks are intimately linked. Deleting the Root Stack deletes all the Nested Stacks.

> [!WARNING] Nested vs. Cross-Stack
> **Only use Nested Stacks when everything shares the same lifecycle**. If sub-components (like a core VPC) need to persist long-term while other components are rapidly created and destroyed, use **Cross-Stack References** instead.

---

## 🔗 Cross-Stack References

Cross-stack references allow completely independent CloudFormation stacks to dynamically share information.

![[CloudFormationCrossStackReferences-1.png]]

![[CloudFormationCrossStackReferences-2.png]]

* **Exports and Imports**: Stack A defines an `Output` and specifies an `Export` directive with a unique name. Stack B then uses the intrinsic function `Fn::ImportValue` (or `!ImportValue`) to import that exact value.
* **Service-Oriented Architecture**: Perfect for scenarios where one stack builds core infrastructure (like an enterprise VPC and subnets) and exports the subnet IDs so that dozens of other entirely disparate application stacks can import them.
* **Unique Names**: The exported name must be globally unique within the specific Region and AWS Account.
* **Protection**: CloudFormation protects exports. You **cannot** delete or change a stack's exported value if another active stack is currently importing it.

---

## 🌍 StackSets

StackSets extend the capability of stacks by enabling you to create, update, or delete infrastructure across **multiple AWS accounts and multiple Regions** with a single operation.

![[CloudFormationStackSets.png]]

* **Core Concept**: You create a StackSet in an Admin account. The StackSet acts as a container. Within it, **Stack Instances** are spawned. These instances represent the physical execution of the stack in target child accounts/regions.
* **Permissions**: Driven either by **Self-Managed** IAM Roles (you manually provision cross-account roles) or **Service-Managed** permissions automatically via AWS Organizations integrations.
* **Fault Tolerance & Concurrency**: You can specify how many accounts to deploy to concurrently, and set a failure tolerance (e.g., if deployment fails in 2 accounts, abort the entire update).
* **Common Use Cases**: Deploying baseline security configurations centrally (e.g., enabling AWS Config, deploying standard IAM roles, or enforcing GuardDuty) across an entire enterprise organisation.

---

## 🗑️ DeletionPolicy

By default, if you delete a logical resource from a template or delete the stack entirely, CloudFormation destroys the underlying physical resources. A `DeletionPolicy` overrides this behaviour on a per-resource basis, preventing accidental data loss.

![[CloudFormationDeletionPolicy.png]]

* **Delete** *(Default)*: Physical resource is destroyed.
* **Retain**: CloudFormation removes the resource from the stack's management but leaves the physical resource untouched and active in AWS.
* **Snapshot**: Automatically takes a final snapshot backup of the resource *before* deleting it. Supported on stateful services: EBS, ElastiCache, Neptune, RDS, and Redshift.

> [!CAUTION] Exam PowerUP
> The `DeletionPolicy` only applies during a **Delete** operation. If you perform an **Update** operation that necessitates replacing the resource (e.g., changing the encryption key of an RDS instance), the old resource is replaced and deleted, and the exact `DeletionPolicy` rules might not prevent data loss depending on the replacement sequence.

---

## 🤝 Stack Roles

By default, CloudFormation executes stack changes using the permissions of the IAM User/Role that clicked the "Create Stack" button.

![[CloudFormationStackRoles.png]]

* **Role Separation**: CloudFormation Stack Roles allow the service to assume a dedicated IAM Role specifically for the deployment process.
* **How It Works**: A developer (Phil) might only have permission to execute CloudFormation templates and explicitly pass the `PassRole` permission for the Stack Role. CloudFormation then assumes that specific Stack Role (which has Admin rights/resource rights) to provision the infrastructure. The developer never needs direct access to the underlying infrastructure.

---

## 🛠️ Bootstrapping: CloudFormationInit, cfn-init & cfn-hup

While EC2 UserData runs bash scripts procedurally (defining *how* to do things line-by-line), `CloudFormationInit` allows you to define configurations declaratively (defining *what* the desired final state is).

![[CloudFormationInit-and-Cfninit.png]]

1. **`AWS::CloudFormation::Init`**: A metadata section defined directly inside the EC2 logical resource in the template. It declares packages to install, files to write, and services to start.
2. **`cfn-init`**: A helper daemon installed on the EC2 instance. During the instance boot cycle (inside UserData), `cfn-init` is called. It talks to CloudFormation, downloads the `AWS::CloudFormation::Init` directives, and applies them to the OS. It is **idempotent** (safe to run multiple times without causing errors).

### Continuous Updates with cfn-hup

![[CloudFormationCfnHUP.png]]

* `cfn-init` fundamentally only runs once at boot. If you update the `AWS::CloudFormation::Init` metadata in the template later, the instance normally ignores it.
* **`cfn-hup`**: A background daemon you can install on the instance. It actively polls the CloudFormation metadata for updates. If you update the template, `cfn-hup` detects the change and automatically re-triggers `cfn-init` to apply the newly desired state to the running instance without needing a reboot!

---

## 🔍 Change Sets

Change Sets allow you to preview exactly what CloudFormation intends to do *before* it actually touches your physical resources.

![[CloudFormationChangeSets.png]]

* **Preview Capabilities**: Shows you if an update will cause **No Interruption**, **Some Interruption** (like a reboot), or a complete **Replacement** (destructive recreation) of physical resources.
* **CI/CD Integration**: Crucial for continuous integration pipelines, allowing automated or manual review of destructive actions before executing the stack update.

---

## 🧩 Custom Resources

CloudFormation doesn't natively support *every* AWS feature immediately, nor does it support third-party systems. Custom Resources bridge this gap.

![[CloudFormationCustomResources.png]]

* **Mechanism**: You define an `AWS::CloudFormation::CustomResource`. During deployment, CloudFormation triggers an external endpoint (usually an **AWS Lambda** function or an **SNS** topic) passing physical resource parameters.
* **Use Cases**: Emptying an S3 bucket before structural deletion, running a complex database migration script, fetching external third-party API configurations, or provisioning resources for an AWS service not yet natively supported by CloudFormation templates.
