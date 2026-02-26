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

---

## 🏰 Bastion Host (Jumpbox)

> [!INFO] Definition
> A "Bastion Host" is a specialized EC2 instance located in a **Public Subnet** used to provide secure management access to instances in **Private Subnets**.

*   **Architecture**:
    - **Inbound**: You connect to the Bastion via SSH (Linux) or RDP (Windows) from the internet.
    - **Outbound (Internal)**: From the Bastion, you jump to private instances using their private IPs.
*   **Security**: Since it is the only way "into" the VPC, it should be heavily hardened and its Security Group should strictly limit inbound traffic to your specific source IP.
*   **Cost**: Use a small instance type (e.g., `t3.nano`) to minimize costs, as it requires very little compute power.

---

---

## 🛡️ Stateful vs. Stateless Firewalls

> [!INFO] Connection Anatomy
> Every network communication (connection) consists of two parts: a **Request** and a **Response**. One goes inbound, while the other goes outbound.

### 🧱 Stateless Firewalls
*   **Behavior**: Does not "remember" the state of a connection. It treats the Request and the Response as two completely independent events.
*   **Rule Requirement**: Requires **two explicit rules** (1 Inbound, 1 Outbound) for every single connection.
*   **Ephemeral Ports**: Because the response comes back on a high-numbered port (e.g., 1024-65535), stateless firewalls usually require opening a wide range of ports for returning traffic.
*   **Overhead**: Higher administrative effort; "clunky" to manage.

### 🧠 Stateful Firewalls
*   **Behavior**: Intelligent enough to track the "state" of a connection. If a request is allowed out, the router automatically remembers this and allows the corresponding response back in.
*   **Rule Requirement**: Only requires **one rule** (the initiating direction).
*   **Overhead**: Significantly lower management overhead and more secure by default (only specific ports are truly "open").

---

## 🥅 Network Access Control Lists (NACL)

> [!INFO] Definition
> A NACL is an optional, **stateless** layer of security for your VPC that acts as a firewall for controlling traffic in and out of one or more **subnets**.

### Core Characteristics
*   **Boundary**: Filters traffic crossing the **Subnet Boundary**. Traffic *within* the same subnet is never seen or impacted by a NACL.
*   **Associations**:
    - Every subnet **must** have an associated NACL.
    - A subnet can be associated with exactly **one** NACL.
    - A single NACL can be associated with **multiple** subnets.
*   **Targeting**: NACLs are not aware of "resources" (like EC2 instances); they only understand **IP addresses**, **CIDRs**, **Protocols**, and **Port Ranges**.

### 🚥 Rule Processing
1.  **Inbound & Outbound**: Each NACL has separate, independent rulesets for traffic entering and leaving the subnet.
2.  **Order Matters**: Rules are processed in numerical order, **lowest number first**.
3.  **Short-Circuiting**: Once a match is found (Allow or Deny), the router stops processing and applies the action immediately.
4.  **The Implicit Deny**: Every NACL ends with an invisible `*` rule (Default Deny) that drops any traffic that didn't match an earlier rule.

![[NACL-1.png]]

> [!TIP] Exam Nugget: The Stateless Trap
> Because NACLs are **stateless**, you must manually account for **Ephemeral Ports**.
> - If you allow inbound HTTP (Port 80), you **must** also allow outbound traffic on ephemeral ports (typically 1024-65535) so the response can reach the client.

### Default vs. Custom NACLs

| Feature | Default NACL (Created with VPC) | Custom NACL (User Created) |
| :--- | :--- | :--- |
| **Initial State** | All traffic <span style="color:rgb(0, 176, 80)">ALLOW</span> (Rule 100: 0.0.0.0/0 ALLOW). | All traffic <span style="color:rgb(255, 0, 0)">DENY</span>. |
| **Associations** | Associated with all VPC subnets by default. | Associated with **no** subnets until manually assigned. |

![[NACL-5.png]]

### 🎯 Best Practices & Use Cases
*   **Block Specific IPs**: Use NACLs to explicitly **Deny** traffic from a specific malicious IP or CIDR block (Security Groups cannot perform "Deny" actions).
*   **Layered Defense**: Use NACLs for broad, subnet-level protection and Security Groups for granular, instance-level security.

---

## 🛡️ VPC Security Groups

> [!INFO] Definition
> A Security Group (SG) acts as a virtual, **stateful** firewall for your **Elastic Network Interfaces (ENIs)** to control inbound and outbound traffic.

### Core Characteristics
*   **Stateful**: SGs track the "state" of connections. If an inbound request is allowed, the outbound response is automatically allowed, regardless of outbound rules (and vice-versa).
*   **Allow-Only**: SGs support **Allow** rules only. There is no such thing as an "Explicit Deny" (use NACLs for that).
*   **Implicit Deny**: Any traffic not specifically allowed by a rule is dropped by the default "deny-all" rule at the end of the ruleset.
*   **Attachment**:
    - **Exam Critical**: SGs are attached to **ENIs (Elastic Network Interfaces)**, not directly to the EC2 instances or subnets.
    - Multiple SGs can be attached to a single ENI (up to 5 by default).

### 🏷️ Logical Referencing (Scaling Power)
Unlike NACLs that only understand IPs/CIDRs, Security Groups can reference other **Security Groups** as a source or destination.
*   **Scalability**: If Security Group A allows traffic from Security Group B, then **any** resource with SG-B attached can talk to **any** resource with SG-A attached on the specified port.
*   **No IP Management**: You don't need to update rules when instances are added or replaced; the firewall rule scales automatically based on the membership of the target SG.

### 🔄 Security Group Self-Referencing

> [!TIP] Definition & Benefit
> **Self-referencing** is a configuration where a Security Group (e.g., `sg-web-servers`) contains a rule that allows inbound traffic from **itself** (the same SG ID).

*   **How it Works**: It treats all members of the Security Group as a "trusted group."
*   **Primary Benefit**: Allows any ENI associated with that SG to communicate with any other ENI associated with the same SG (e.g., all instances in a cluster can talk to each other).
*   **Scaling**: As you launch new instances into the group, they are immediately granted access to the rest of the cluster members without needing any IP-based rule updates.

![[SG-4.png]]

---
