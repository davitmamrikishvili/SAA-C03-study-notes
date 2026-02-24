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

## 🌐 Subnets

> [!INFO] Definition
> A subnet is a sub-range of IP addresses within a VPC. It allows you to group resources based on security and operational needs.

*   **Regional Scope**: A VPC spans an entire region, but a **Subnet exists within exactly one Availability Zone (AZ)**.
*   **Isolation**: One subnet's CIDR block **cannot overlap** with any other subnet in the same VPC.
*   **Public vs. Private**:
    *   **Public Subnet**: Has a route to an **Internet Gateway (IGW)** in its route table.
    *   **Private Subnet**: Does **not** have a direct route to an IGW.

### 🛡️ Reserved IP Addresses
AWS reserves the **first four** and the **last one** IP address in every subnet (5 total). For a CIDR `10.0.0.0/24`:
1.  `10.0.0.0`: **Network Address**.
2.  `10.0.0.1`: **VPC Router** (Default Gateway).
3.  `10.0.0.2`: **Amazon Provided DNS** (Route 53 Resolver).
4.  `10.0.0.3`: **Future Use** (Reserved for AWS).
5.  `10.0.0.255`: **Network Broadcast** (AWS does not support broadcast, but the address is reserved).

### ⚙️ IP Allocation Settings
*   **Auto-assign Public IPv4**: If enabled, instances launched in this subnet automatically receive a public IPv4 address. Generally enabled for Public subnets.
*   **Auto-assign IPv6**: If enabled, instances automatically receive an IPv6 address from the subnet's GUA (Global Unicast Address) range.

---

## Fundamental Components
* **Internet Gateway (IGW)**: Allows communication between your VPC and the internet.
* **Virtual Private Gateway (VPG)**: Used for VPN connections.
* **NAT Gateway**: Allows instances in a private subnet to connect to the internet (one-way) while preventing the internet from initiating connections.
* **Security Groups**: Stateful firewalls for **instances**.
* **NACLs (Network Access Control Lists)**: Stateless firewalls for **subnets**.

---

## 🏗️ VPC Sizing and Structure
*   **Minimum Size**: `/28` (16 IP addresses).
*   **Maximum Size**: `/16` (65,536 IP addresses).
*   **CIDR Flexibility**: You can add **Secondary CIDR blocks** to a VPC after creation if you run out of space, but you cannot change the primary CIDR block.

---

## 📛 DNS in a VPC
AWS provides a built-in DNS resolver called the **Route 53 Resolver** (formerly "AmazonProvidedDNS").

*   **Resolver Address**: Always the VPC **Base IP + 2** (e.g., in `10.0.0.0/16`, the DNS is at `10.0.0.2`).
*   **`enableDnsSupport`**:
    - **Description**: Enables/Disables the Amazon DNS server.
    - **Impact**: If `false`, the Route 53 Resolver is inaccessible, and instances cannot resolve AWS hostnames or external domains without a custom DNS server.
*   **`enableDnsHostnames`**:
    - **Description**: Determines if instances receive public DNS hostnames.
    - **Impact**: If `true`, instances with public IPs are assigned a public DNS name (e.g., `ec2-54...`). Requires `enableDnsSupport` to be `true`.

---

## 🛠️ DHCP Options Set
A configuration object that defines network settings for instances in the VPC (e.g., domain names, custom DNS servers, NTP servers).

*   **Immutability**: You **cannot edit** a DHCP Options Set. To make changes, you must:
    1.  Create a **new** DHCP Options Set.
    2.  Associate it with the VPC.
*   **Persistence**: Once associated, the settings flow down to all subnets and instances in the VPC.

---
*Next Topic: Subnetting & Route Tables*