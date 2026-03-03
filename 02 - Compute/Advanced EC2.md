---
tags:
  - aws/ec2
  - compute
  - automation
category: Compute
---

# 🚀 Advanced EC2

## 📥 EC2 User Data & Bootstrapping
> [!INFO] Definition
> **Bootstrapping** is the process of automatically configuring an EC2 instance or installing software when it first launches. This is achieved using **EC2 User Data**.

### 🛠️ Technical Mechanics
* **Execution**: User Data scripts are executed by the instance OS (e.g., `cloud-init` for Linux) **only during the initial launch**.
* **Opaque to AWS**: EC2 does not interpret the script. It simply passes the block of data to the OS. If the script fails, the instance will still show as "Running" (Silent Failure).
* **Accessibility**: User Data is injected into the instance and accessed via the Metadata IP: `http://169.254.169.254/latest/user-data`.
* **Persistence**: User Data can be modified while the instance is **stopped**, but it will not automatically re-run upon starting unless the OS is configured to do so.

### ⚠️ Constraints & Security
* **Size Limit**: Limited to **16 KB**.
* **Security**: User Data is **NOT secure**. It is stored in plain text. Never use it to pass long-term credentials (IAM roles are the solution for this).
* **Visibility**: Anyone with access to the instance OS or the EC2 Console can view the User Data.

---

## ⏱️ Boot-Time-To-Service-Time
This identifies how long it takes from the moment you request an instance until it is ready to serve traffic.

* **Provisioning Time**: The time AWS takes to allocate hardware and start the VM.
* **Post-Launch Time**: The time taken for software updates, installations, and configurations within the OS.

### 🍱 The Optimal Strategy: Baking vs. Bootstrapping
To minimize "Boot-Time-To-Service-Time," architects use a hybrid approach:

| Strategy          | When to Use                                                                             | Advantage                                                                 |
| :---------------- | :-------------------------------------------------------------------------------------- | :------------------------------------------------------------------------ |
| **AMI Baking**    | **Time-Intensive** tasks (Large app installs, OS hardening, base updates).              | Very fast launch times; everything is pre-installed.                      |
| **Bootstrapping** | **Dynamic** tasks (Applying final config, joining a cluster, fetching latest app code). | Extreme flexibility; the same AMI can be used for different environments. |

![[EC2Bootstrapping-2.png]]

> [!TIP] Exam PowerUP
> **Bake** part of the installation process that is time-intensive, **Bootstrap** the remaining dynamic tasks.

---

## 🛠️ CloudFormation Init (cfn-init)

While User Data is powerful, it is purely **procedural** and "fire-and-forget." For more complex, declarative configuration management, architects use **CloudFormation Init**.

### 🧩 Procedural (User Data) vs. Desired State (cfn-init)
* **User Data**: A script that runs line-by-line. If a command fails, the script continues (unless you handle errors manually). It is difficult to update or maintain.
* **cfn-init**: A configuration management system built into CloudFormation. You define the **Desired State** (what the instance *should* look like), and the `cfn-init` helper script makes it so.

### 🏗️ The 7 Sections of cfn-init
The `AWS::CloudFormation::Init` resource allows you to manage:
1. **Packages**: Install/Update software packages (RPM, MSI, Yum, etc.).
2. **Groups**: Create OS-level user groups.
3. **Users**: Create OS-level users.
4. **Sources**: Download and extract archives (tar, zip) into directories.
5. **Files**: Create files with specific content/permissions.
6. **Commands**: Execute shell commands in a specific order.
7. **Services**: Start/Stop/Disable OS services (ensure a service is always running).

### 🔄 How it Works
1. **Directives**: Configuration is stored in the **Metadata** section of the CloudFormation template under `AWS::CloudFormation::Init`.
2. **Trigger**: You call the `cfn-init` helper script from within the **User Data**.
3. **Execution**: `cfn-init` connects to CloudFormation, retrieves the metadata, and applies the configuration to the local instance.
4. **Updates**: Unlike User Data, `cfn-init` can be re-run during **Stack Updates** to modify the instance configuration without a total replacement.
![[CFN-INIT-1.png]]

---

## 🚦 CloudFormation Wait Conditions

To ensure an instance is strictly "Ready" before CloudFormation proceeds (e.g., before updating a Load Balancer), we use `cfn-signal` and a `CreationPolicy`.

### 📡 cfn-signal
* **Purpose**: A helper script used to send a "Success" or "Failure" signal back to CloudFormation.
* **Usage**: Typically placed at the very end of your User Data or `cfn-init` command list. It tells AWS: "The software is installed and the service is running."

### 🛡️ CreationPolicy
* **Purpose**: Tells the CloudFormation stack to **wait** for a specific number of signals before marking the resource as `CREATE_COMPLETE`.
* **Timeout**: You define a timeout period (e.g., 5 minutes). If the `cfn-signal` isn't received within that time, the stack fails and rolls back.

> [!TIP] Exam PowerUP: The Duo
> Always think of **cfn-signal** and **CreationPolicy** as a pair. One sends the signal (from the instance), and the other waits for it (in the CFN template). This prevents your stack from appearing "finished" while your application is still installing in the background.

![[CFN-INIT-2.png]]

---

## 🔑 EC2 Instance Roles

EC2 Instance Roles are the secure, best-practice way of providing AWS permissions to applications running on an EC2 instance without managing long-term credentials.

### 🏗️ Architecture: Role vs. Instance Profile
* **IAM Role**: The identity itself. It contains a **Permissions Policy** (what it can do) and a **Trust Policy** (which services, like `ec2.amazonaws.com`, can assume it).
* **Instance Profile**: A logical "container" or "wrapper" for the IAM Role. This is what you actually attach to the EC2 instance.
> [!NOTE] Console vs. CLI
>  When using the AWS Console, an Instance Profile is created and named automatically for you. When using the CLI or CloudFormation, you often have to manage them as separate resources.

### 📡 How it Works (Credential Delivery)
1. **Assumption**: The EC2 service assumes the role attached via the Instance Profile.
2. **IMDS Delivery**: Temporary security credentials (Access Key, Secret Key, and Session Token) are delivered into the instance via the **Instance Metadata Service (IMDS)**.
3. **Endpoint**: `http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>`
4. **Automatic Renewal**: AWS automatically rotates these credentials. Applications using the AWS SDK or CLI will automatically fetch and refresh these credentials from the metadata endpoint.

### 🛡️ Security Best Practices
* **Never Use Static Keys**: Never store `AWSAccessKeyId` or `AWSSecretAccessKey` in configuration files or Environment Variables on an EC2 instance.
* **Least Privilege**: Attach only the specific permissions required for the application.
* **Universal Support**: Most AWS-aware software and all official SDKs will automatically check for metadata credentials if no other credentials are found.

> [!TIP] Exam PowerUP: Troubleshooting Permissions
> If an application on EC2 gets an "Access Denied" error, check:
> 1. Is there an **Instance Profile** attached?
> 2. Does the **IAM Role** have the correct permissions?
> 3. Does the **Trust Policy** allow `ec2.amazonaws.com`?