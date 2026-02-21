---
tags:
  - aws/ec2
  - compute
category: Compute
---

# EC2 - Elastic Compute Cloud

> [!INFO] Definition
> Amazon EC2 provides scalable computing capacity in the AWS Cloud. It is an **Infrastructure as a Service (IaaS)**.

## Core Characteristics
* **Virtual Machines**: Known as **Instances**.
* **Regional & AZ Resilient**: Instances are launched into a specific **Availability Zone**. If the AZ fails, the instance fails.
* **Storage**:
	* **EBS (Elastic Block Store)**: Network storage, persists data.
	* **Instance Store**: Local physical storage, ephemeral (data lost on stop/terminate).

## Instance Lifecycle
* **Running**: Active and billing.
* **Stopped**: Data on EBS persists, but you aren't billed for compute (only for EBS storage).
* **Terminated**: Instance is deleted and cannot be restarted.

## Security & Access
* **Security Groups**: Virtual firewalls at the instance level.
* **Key Pairs**: Used for secure login (SSH for Linux, RDP for Windows).
	* **SSH**: Port 22
	* **RDP**: Port 3389

## Metadata
* **Instance Metadata Service (IMDS)**: Accessible via `http://169.254.169.254/latest/meta-data/`. Used to get info about the instance from within it.