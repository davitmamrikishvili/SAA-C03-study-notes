---
tags:
  - aws/vpc
  - networking
category: Networking & Content Delivery
---

# VPC Core & Fundamental Components

> [!INFO] Definition
> A VPC is a private, isolated virtual network within the AWS cloud. It belongs to a single AWS Account and is hosted within a single **AWS Region**.

## Core Characteristics
* **Isolation**: By default, VPCs are private and logically isolated from each other.
* **Scope**: A VPC spans all Availability Zones (AZs) within a region.
* **Types**:
	* **Default VPC**: Created automatically in every region. Has a `/20` subnet in each AZ and an Internet Gateway (IGW) pre-attached.
	* **Custom VPC**: Defined by the user with specific CIDR blocks and networking components.

---

## 🏗️ VPC Sizing and Structure
* **Minimum Size**: `/28` (16 IP addresses).
* **Maximum Size**: `/16` (65,536 IP addresses).
* **CIDR Flexibility**: You can add **Secondary CIDR blocks** to a VPC after creation if you run out of space, but you cannot change the primary CIDR block.

---

## 📛 DNS in a VPC
AWS provides a built-in DNS resolver called the **Route 53 Resolver** (formerly "AmazonProvidedDNS").

* **Resolver Address**: Always the VPC **Base IP + 2** (e.g., in `10.0.0.0/16`, the DNS is at `10.0.0.2`).
* **`enableDnsSupport`**:
    - **Description**: Enables/Disables the Amazon DNS server.
    - **Impact**: If `false`, the Route 53 Resolver is inaccessible, and instances cannot resolve AWS hostnames or external domains without a custom DNS server.
* **`enableDnsHostnames`**:
    - **Description**: Determines if instances receive public DNS hostnames.
    - **Impact**: If `true`, instances with public IPs are assigned a public DNS name (e.g., `ec2-54...`). Requires `enableDnsSupport` to be `true`.

---

## 🛠️ DHCP Options Set
A configuration object that defines network settings for instances in the VPC (e.g., domain names, custom DNS servers, NTP servers).

* **Immutability**: You **cannot edit** a DHCP Options Set. To make changes, you must:
    1. Create a **new** DHCP Options Set.
    2. Associate it with the VPC.
* **Persistence**: Once associated, the settings flow down to all subnets and instances in the VPC.

---

## 📂 Sub-Topics
For more detailed information on specific VPC components, see the following notes:

1. **[[VPC Subnets & Routing]]**: Subnet types, Route Tables, and Internet Gateways.
2. **[[VPC Security (NACLs & SGs)]]**: Security Groups, NACLs, and Bastion Hosts.
3. **[[VPC NAT & Connectivity]]**: NAT Gateways, NAT Instances, and Egress-only IGWs.