---
tags:
  - aws/vpc
  - networking
category: Networking & Content Delivery
---

# VPC Subnets & Routing

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

## 🗺️ VPC Routing

> [!INFO] The VPC Router
> The VPC Router is a highly available, managed software function that handles traffic between subnets. It runs in every AZ used by the VPC and is accessible at the **Base IP + 1** of every subnet.

### Route Tables (RT)
*   **Customization**: You control the router by defining **Route Tables**.
*   **Associations**:
    - One subnet can be associated with exactly **one** Route Table at a time.
    - One Route Table can be associated with **multiple** subnets.
*   **Main Route Table**: Every VPC has a "Main" RT. If a subnet is not explicitly associated with a custom RT, it uses the Main RT by default.
*   **The "Local" Route**: Every RT contains a default local route for the VPC's CIDR ranges (IPv4/IPv6).
    - **Immutability**: This route cannot be deleted or modified.
    - **Priority**: Local routes always take precedence over more specific routes to ensure internal VPC connectivity.

---

## 🚪 Internet Gateway (IGW)

> [!INFO] Definition
> A horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet.

*   **Regional Scope**: One IGW serves all AZs in a Region.
*   **Attachments**:
    - A VPC can have at most **one** IGW attached.
    - An IGW can be attached to exactly **one** VPC at a time.
*   **Function**: Acts as a bridge between the VPC and the **AWS Public Zone** (Internet, S3, SQS, etc.).

### 🛠️ Creating a Public Subnet (Workflow)
To make a subnet "Public" (capable of 2-way internet traffic), follows these steps:
1.  **Create** an Internet Gateway (IGW).
2.  **Attach** the IGW to your VPC.
3.  **Create** a custom Route Table.
4.  **Add a Route**: Set the destination to `0.0.0.0/0` (and `::/0` for IPv6) with the Target as the **IGW**.
5.  **Associate** this Route Table with the desired subnet.
6.  **Enable IP Allocation**: Ensure the subnet is configured to auto-assign public IPv4/IPv6 addresses.

---

## 🔍 How Addressing Works (Public vs. Private)

### 🧩 IPv4 Strategy (Network Address Translation)
*   **OS Level**: An EC2 instance's Operating System **only sees its Private IP**. It is completely unaware of its Public IP.
*   **IGW Level**: The IGW maintains a 1-to-1 mapping (Static NAT) between the instance's private IP and its public IP. When traffic leaves the VPC via the IGW, the source IP is translated to the Public IP.

### 🌐 IPv6 Strategy (Native Routing)
*   **OS Level**: Unlike IPv4, all VPC IPv6 addresses are **Global Unicast Addresses** (publicly routable). The instance Operating System sees and configures the public IPv6 address directly.
*   **IGW Level**: No translation (NAT) occurs. The IGW simply passes the traffic through as-is.
