---
tags:
  - aws/ec2
  - compute
  - monitoring
category: Compute
---

# 🚦 EC2 Instance Recovery & Protection

Every EC2 instance undergoes two automated status checks. If both pass, you see **"2/2 checks passed"**. During launch, you will see **"Initializing"**.

## 🛠️ The 2/2 Status Checks
| Check Type | Target | Common Failure Causes |
| :--- | :--- | :--- |
| **System Status** | **The Underlying Host** | Loss of power, loss of network connectivity, hardware/software issues on the physical host. |
| **Instance Status** | **The Virtual Machine / OS** | Corrupted filesystem, incorrect networking config, OS kernel issues, or incompatible drivers. |

> [!TIP] Troubleshooting
> * **System Failure**: Usually requires a **Stop & Start** to force the instance onto a new, healthy host.
> * **Instance Failure**: Usually requires a **Reboot** or manual OS configuration fix.

---

## 🔄 Auto-Recovery (CloudWatch Alarms)
You can automate the response to a **System Status Check failure** using a CloudWatch Alarm.

* **How it works**: If the system check fails, AWS automatically moves the instance to a **new physical host**.
* **What is Preserved?**: The instance retains its Instance ID, Private IP addresses, Elastic IP addresses, and all metadata.
* **The Instance Store Limitation**: Auto-recovery **will not work** if the instance has any **Instance Store volumes** attached, as those are physically tied to the original (failed) host.

---

## 🛡️ Termination Protection
* **The Mechanism**: Setting the `disableApiTermination` attribute to `true`.
* **Purpose**: Prevents the instance from being terminated via the AWS Console, CLI, or SDK accidentally.
* **How to Terminate**: You must first manually disable the protection before the terminate command will succeed.
* **Role Separation**: Useful for protecting a production database where one admin manages "Security/Policies" (enabling protection) and another manages "Operations" (preventing accidental deletions).

---

## ⏹️ Shutdown Behavior
This defines what happens when you initiate a shutdown **from within the Operating System** (e.g., `sudo shutdown -h now`).

* **Stop (Default)**: The instance moves to the `stopped` state. You can start it again later.
* **Terminate**: The instance is permanently deleted as soon as the OS shuts down. Useful for temporary batch processing instances to ensure you stop paying immediately.
