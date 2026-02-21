---
tags:
  - aws/vpc
  - networking
category: Networking & Content Delivery
---

# VPC - Virtual Private Cloud

> [!INFO] Definition
> A VPC is a private, isolated virtual network within the AWS cloud. It belongs to a single AWS Account and is hosted within a single **AWS Region**.

## Core Characteristics
* **Isolation**: By default, VPCs are private and logically isolated from each other.
* **Scope**: A VPC spans all Availability Zones (AZs) within a region.
* **Types**:
	* **Default VPC**: Created automatically in every region. Has a `/20` subnet in each AZ and an Internet Gateway (IGW) pre-attached.
	* **Custom VPC**: Defined by the user with specific CIDR blocks and networking components.

## Subnets
* A subnet is a range of IP addresses in your VPC.
* **Scope**: A subnet resides within **one Availability Zone**.
* **Public Subnet**: Has a route to an Internet Gateway (IGW).
* **Private Subnet**: Does not have a direct route to an IGW.

## Fundamental Components
* **Internet Gateway (IGW)**: Allows communication between your VPC and the internet.
* **Virtual Private Gateway (VPG)**: Used for VPN connections.
* **NAT Gateway**: Allows instances in a private subnet to connect to the internet (one-way) while preventing the internet from initiating connections.
* **Security Groups**: Stateful firewalls for **instances**.
* **NACLs (Network Access Control Lists)**: Stateless firewalls for **subnets**.

## IPv4 vs IPv6
* VPCs primarily use IPv4 (CIDR blocks).
* AWS Reserved IPs: The first 4 and the last 1 IP addresses in every subnet are reserved by AWS.

---
*Next Topic: Subnetting & Route Tables*
