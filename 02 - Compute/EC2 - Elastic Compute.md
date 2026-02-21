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

## EC2 Instance Types
Instances differ by CPU, Memory, Storage (Capacity/Type), Resource Ratios, and Network Bandwidth.

### 🏷️ Decoding the Names
Example: `<span style="color:rgb(255, 192, 0)">R</span><span style="color:rgb(255, 0, 0)">5</span><span style="color:rgb(0, 176, 240)">dn</span>.<span style="color:rgb(240, 75, 200)">8xlarge</span>`
* **<span style="color:rgb(255, 192, 0)">Instance Family</span>**: (e.g., **R** for Memory)
* **<span style="color:rgb(255, 0, 0)">Instance Generation</span>**: (e.g., **5**)
* **<span style="color:rgb(0, 176, 240)">Additional Capabilities</span>**: (e.g., **d** for Disk, **n** for Network)
* **<span style="color:rgb(240, 75, 200)">Instance Size</span>**: (e.g., **8xlarge**)

### 📁 Instance Categories
| Category | Types | Use Cases |
| --- | --- | --- |
| **General Purpose** | T3, M5, A1 | Diverse workloads, balanced resources. |
| **Compute Optimized** | C5, C6g | Media processing, HPC, Scientific modeling, Gaming. |
| **Memory Optimized** | R5, X1, z1d | High-performance databases, Real-time analytics, In-memory caches. |
| **Accelerated Computing** | P3, G4, F1, Inf1 | Machine Learning, GPU intensive, Genomics (FPGA). |
| **Storage Optimized** | I3, D2, H1 | NoSQL, Data warehousing, HDFS, Kafka. |

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
* **Instance Metadata Service (IMDS)**: Accessible via `http://169.254.169.254/latest/meta-data/`. Used to query information about the instance (e.g., public IP, instance-id) from within the OS.
