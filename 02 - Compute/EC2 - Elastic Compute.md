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
Example: <span style="color:rgb(255, 192, 0)">R</span><span style="color:rgb(255, 0, 0)">5</span><span style="color:rgb(0, 176, 240)">dn</span>.<span style="color:rgb(240, 75, 200)">8xlarge</span>
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

---

## 🌐 EC2 Networking & DNS Architecture

> [!INFO] Elastic Network Interface (ENI)
> An ENI represents a virtual network card. It is the logical component in a VPC that connects your instances to the network.

### 🧬 ENI Characteristics
* **Primary ENI (eth0)**: Automatically created and attached when an instance is launched. You **cannot detach** the primary ENI.
* **Secondary ENIs**: Can be manually created and attached/detached while the instance is running.
    * **AZ Boundary**: An ENI must be in the same Availability Zone (AZ) as the instance, but it can be in a **different subnet**.
* **Components of an ENI**:
    * **MAC Address**: A unique hardware identifier visible to the Guest OS.
    * **Primary Private IPv4**: Fixed for the life of the ENI.
    * **Secondary Private IPv4s**: Multiple IPs can be assigned to a single ENI.
    * **Public IPv4**: 0 or 1 Public IP (assigned by AWS or via Elastic IP).
    * **Elastic IP (EIP)**: A static, public IPv4 address that can be moved between ENIs/instances.
    * **IPv6 Addresses**: 0 or more.
    * **Security Groups**: SGs are attached to the **ENI**, not the instance.

![[EC2networking-2.png]]

### 🛡️ Source/Destination Check
> [!WARNING] Important for NAT/Firewalls
> By default, every ENI performs a "Source/Destination Check." It only allows traffic if the instance is the **source** or the **intended destination** of the packet.

* **When to Disable**: You must **disable** this check for any instance acting as a "middleman" (Router, NAT Instance, or Firewall) so it can forward packets that aren't addressed to it.

### 🎨 IP Addressing & DNS Behavior

#### 🧩 Public vs. Private IPs
* **OS Visibility**: The Guest Operating System **never sees its Public IP**. It only knows about its Private IP.
* **Dynamic Nature**: Public IPv4 addresses are **dynamic**. If you STOP and START an instance, it receives a **new** public IP. (Note: Elastic IPs are static and solve this).

#### 📍 DNS Resolution Logic
* **Public DNS Name**: AWS provides a public DNS name (e.g., `ec2-54...`).
* **Internal Resolution**: If queried from within the same VPC, the public DNS resolves to the **Private IP** (saving bandwidth and lowering latency).
* **External Resolution**: If queried from outside the VPC (the internet), it resolves to the **Public IP**.

### 🚀 Exam PowerUP: ENI Use Cases

1. **Licensing Management**: Some legacy software is "locked" to a specific MAC address. By using a **Secondary ENI**, you can detach it from an old instance and move it to a new one, keeping the MAC address (and the license) intact.
2. **Management Networks**: Use multiple ENIs to create "Multi-homed" instances. For example, `eth0` for public data traffic and `eth1` for a secure, private management/logging subnet.
3. **The Elastic IP Trap**: If an instance has a standard public IPv4 and you assign an **Elastic IP**, you lose the original IP forever. If you later remove the EIP, the instance will get a *brand new* dynamic public IP, not the original one.

---

## 💰 EC2 Purchase Options

### 🟢 On-Demand
* **Most Flexible**: The default choice. No long-term commitment and no upfront cost.
* **Isolation**: Multiple customer instances run on shared hardware, but are logically isolated.
* **Billing**: Pay-per-second while running. (EBS still bills when instance is stopped).
* **Best For**: Short-term workloads, unpredictable traffic, or testing new applications.
* **Constraint**: No capacity reservation (you might get an "Insufficient Capacity" error during peak times).

![[EC2PurchaseOptions-1.png]]

### 🟡 Spot Instances
* **Deepest Discount**: Up to **90% cheaper** than On-Demand by using spare AWS capacity.
* **Variable Pricing**: The "Spot Price" fluctuates based on demand/supply.
* **Interruption**: If AWS needs the capacity back or the spot price exceeds your max bid, you get a **2-minute warning** before the instance is terminated.
* **Best For**: Stateless, fault-tolerant, or flexible workloads (Batch jobs, Data analysis, CI/CD).
* **Rule of Thumb**: Never use Spot for stateful workloads that cannot tolerate interruption.

![[EC2PurchaseOptions-2.png]]

### 🔵 Reserved Instances (RI)
* **Commitment**: 1 or 3-year term for significant discounts.
* **Scope vs. Capacity**:
    * **Zonal RI**: Locked to a specific AZ. **Reserves capacity** but only applies discount in that AZ.
    * **Regional RI**: Applies to any AZ in the region. **No capacity reservation**, but more flexible for scaling.
* **Payment Options**:
    * **All Upfront**: Highest discount.
    * **Partial Upfront**: Balanced.
    * **No Upfront**: Lowest discount for agreeing to the term.
* **Standard vs. Convertible**: Standard RIs have higher discounts but cannot change instance families; Convertibles allow family changes.

![[EC2PurchaseOptions-3.png]]

### 🛡️ Dedicated Hosts
* **Full Hardware Control**: A physical server is allocated entirely to you.
* **Billing**: You pay for the **Host**, not the instances running on it.
* **Licensing (BYOL)**: Essential for software licensed by **physical sockets/cores** (e.g., Oracle, Windows Server).
* **Host Affinity**: You can ensure specific instances always run on the same physical hardware.

![[EC2PurchaseOptions-4.png]]

### 🏢 Dedicated Instances
* **Hardware Isolation**: Instances run on hardware that is physically separated from other AWS customers.
* **Difference from Hosts**: You don't get control over the physical server attributes (sockets/cores). You just guarantee your neighbors are *you*.
* **Cost**: Includes an additional hourly fee per region where they are used.

![[EC2PurchaseOptions-5.png]]

*Continue from 3:56:21 
