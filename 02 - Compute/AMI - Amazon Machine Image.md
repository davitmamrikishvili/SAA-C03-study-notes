---
tags:
  - aws/ec2
  - compute
category: Compute
---

# AMI - Amazon Machine Image

> [!INFO] Definition
> An AMI is a template that contains the software configuration (operating system, application server, and applications) required to launch an instance.

## Key Facts
* **Regional**: AMIs are stored in a specific region. To use an AMI in another region, you must **copy** it. Each AMI has unique ID.
* **Permissions**:
	* **Public**: Available to all AWS accounts.
	* **Explicit**: Shared with specific AWS accounts.
	* **Implicit (Owner)**: Only available to the owner.

## AMI Components
* **Root Volume Template**: (EBS snapshot or Instance Store template).
* **Block Device Mapping**: Defines which volumes to attach when the instance is launched.
* **Launch Permissions**: Who can use the AMI.

## Custom AMIs
* You can "Bake" an AMI by launching an instance, configuring it, and then creating an image from it to save time during scaling.

## 🛒 AMI Sources
1.  **AWS Provided / Quick Start**: Clean, pre-hardened OS images (e.g., Amazon Linux 2023).
2.  **Community AMIs**: Publicly shared images (use with caution).
3.  **AWS Marketplace**: Pre-configured software stacks (e.g., WordPress, Firewalls). May include commercial software costs.
4.  **Custom AMIs**: Built and owned by you.

---

## 🔄 The AMI Lifecycle (Baking Process)

> [!INFO] Definition: "Baking" an AMI
> Baking is the process of creating a fully pre-configured "Golden Image" so that future instances can launch ready-to-run without manual setup or slow boot scripts.

1.  **Launch**: Use an existing AMI to launch a temporary instance.
2.  **Configure**: Install your application, security patches, and required libraries.
3.  **Create Image**: Trigger the "Create Image" action on the running instance.

### 🧩 What happens during "Create Image"?
When you create an AMI, AWS performs several background steps:
*   **Snapshots**: It automatically takes a snapshot of **every EBS volume** attached to the instance.
*   **Metadata**: It records the instance configuration (CPU type, architecture, RAM requirements).
*   **Block Device Mapping**: It creates a data table that links the new **Snapshot IDs** to their original **Device IDs** (e.g., `snap-12345` $\to$ `/dev/xvda`).

---

## 🚀 Exam PowerUP: AMI Mechanics

*   **Immutable**: An AMI cannot be edited. To update it, you must launch a new instance from it, change the config, and bake a **new** AMI.
*   **Cross-Region Copy**: Because AMIs are **Regional**, you must copy them to other regions before they can be used there. This process creates a new AMI ID in the target region.
*   **Cost Efficiency**:
    - You are **not** billed for the AMI itself.
    - You are billed for the **storage capacity** used by the EBS snapshots that the AMI references.
*   **Launch Permissions**: You can keep your AMI private, share it with specific AWS Account IDs, or make it public to everyone.

---
*Next Topic: EC2 Storage (Instance Store vs. EBS)*