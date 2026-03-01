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
