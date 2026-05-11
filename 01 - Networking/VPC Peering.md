---
tags:
  - aws/vpc
  - networking
category: Networking
---

# 🔗 VPC Peering

> [!INFO] Definition
> **VPC Peering** creates a direct, private, encrypted network link between two VPCs. Traffic between peered VPCs routes through the AWS global network rather than the public internet.

---

## 🏗️ Architecture

When you create a VPC Peering connection, AWS creates a **logical gateway object** inside each of the two VPCs. Like any gateway, routing must be explicitly configured — you need to add routes in each VPC's route tables pointing the other VPC's CIDR to the peering connection.

* **Security Groups & NACLs** can still be used to filter traffic flowing across the peering connection.

![[VPCPeering.png]]

---

## 🌐 Scope & Flexibility

* **Same or Cross-Region**: Peering works between VPCs in the same AWS region or across different regions.
* **Same or Cross-Account**: Works within the same AWS account or between two different AWS accounts.
* **Encrypted Communication**: Cross-region peering traffic is encrypted and transits over the AWS private global network.

---

## ⚙️ Key Behaviours & Configuration

* **DNS Resolution**: When creating a peer, you can enable an option so that the **public DNS hostnames** of services in peered VPCs resolve to their **private internal IP addresses**. This allows you to use the same DNS names regardless of which VPC you're connecting from.
* **Security Group References**:
    * **Same-region peers**: You can reference the **Security Group ID** of a resource in the peered VPC directly in your SG rules — no need to use IP ranges.
    * **Cross-region peers**: You must reference the other VPC using **IP addresses or CIDR ranges**.

---

## ⚠️ Critical Limitations

> [!CAUTION] Exam PowerUP: The Two Hard Rules
> 1. **One peering connection = Two VPCs only.** A single peering connection can *never* link more than two VPCs.
> 2. **No Transitive Peering.** If VPC-A peers with VPC-B, and VPC-B peers with VPC-C, VPC-A **cannot** reach VPC-C through VPC-B. Each pair of VPCs that needs to communicate requires its own dedicated peering connection.
> 3. **No Overlapping CIDRs.** A peering connection cannot be established if the two VPCs have overlapping IP address ranges. This must be planned at VPC design time.
