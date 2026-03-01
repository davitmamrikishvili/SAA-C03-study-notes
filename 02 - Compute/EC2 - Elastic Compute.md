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
* **AZ Affinity Limitation**:
	> [!WARNING] Important
	> An EC2 instance running in a specific AZ **cannot** connect to an EBS volume in a different AZ.

## What's EC2 Good for?
* Traditional OS + Application Compute
* Long-running compute & Server-style applications
* Burst or steady-state loads
* Monolithic application stacks
* Migrated application workloads or Disaster Recovery (DR)

## Security & Access
* **Security Groups**: Virtual firewalls at the instance level.
* **Key Pairs**: Used for secure login (SSH for Linux, RDP for Windows).
	* **SSH**: Port 22
	* **RDP**: Port 3389

---

## 📂 EC2 Sub-Topics
Detailed notes on EC2 features have been split into the following sections:

1. **[[EC2 Instance Types & Lifecycle]]**: Decoding names, categories, and the life of an instance.
2. **[[EC2 Networking & DNS]]**: ENIs, IP addressing, DNS resolution, and Source/Destination check.
3. **[[EC2 Pricing & Purchase Options]]**: On-Demand, Spot, RI, Dedicated Hosts, and Savings Plans.
4. **[[EC2 Instance Recovery & Protection]]**: Status Checks, Auto-Recovery, and Termination Protection.
5. **[[AMI - Amazon Machine Image]]**: Pre-configured templates for launching instances.
6. **[[EC2 Scaling (Vertical vs. Horizontal)]]**: Vertical vs. Horizontal scaling trade-offs.
7. **[[EC2 Instance Metadata (IMDS)]]**: Accessing and securing instance metadata (IMDSv1 vs. IMDSv2).
8. **[[Advanced EC2]]** : Advanced EC2 features.