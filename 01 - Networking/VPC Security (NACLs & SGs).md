---
tags:
  - aws/vpc
  - networking
  - security
category: Networking & Content Delivery
---

# VPC Security (NACLs & SGs)

## 🛡️ Stateful vs. Stateless Firewalls

> [!INFO] Connection Anatomy
> Every network communication (connection) consists of two parts: a **Request** and a **Response**. One goes inbound, while the other goes outbound.

### 🧱 Stateless Firewalls
*   **Behavior**: Does not "remember" the state of a connection. It treats the Request and the Response as two completely independent events.
*   **Rule Requirement**: Requires **two explicit rules** (1 Inbound, 1 Outbound) for every single connection.
*   **Ephemeral Ports**: Because the response comes back on a high-numbered port (e.g., 1024-65535), stateless firewalls usually require opening a wide range of ports for returning traffic.

### 🧠 Stateful Firewalls
*   **Behavior**: Intelligent enough to track the "state" of a connection. If a request is allowed out, the router automatically remembers this and allows the corresponding response back in.
*   **Rule Requirement**: Only requires **one rule** (the initiating direction).

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

---

## 🛡️ VPC Security Groups

> [!INFO] Definition
> A Security Group (SG) acts as a virtual, **stateful** firewall for your **Elastic Network Interfaces (ENIs)** to control inbound and outbound traffic.

### Core Characteristics
*   **Stateful**: SGs track the "state" of connections. If an inbound request is allowed, the outbound response is automatically allowed.
*   **Allow-Only**: SGs support **Allow** rules only. There is no such thing as an "Explicit Deny".
*   **Implicit Deny**: Any traffic not specifically allowed by a rule is dropped.
*   **Attachment**:
    - **Exam Critical**: SGs are attached to **ENIs (Elastic Network Interfaces)**, not directly to the EC2 instances.

### 🏷️ Logical Referencing (Scaling Power)
Security Groups can reference other **Security Groups** as a source or destination.
*   **Scalability**: If SG-A allows traffic from SG-B, then any resource with SG-B attached can talk to SG-A.
*   **No IP Management**: Firewall rules scale automatically based on SG membership.

### 🔄 Security Group Self-Referencing
*   **How it Works**: A rule that allows inbound traffic from **itself** (the same SG ID).
*   **Benefit**: Allows any ENI associated with that SG to communicate with any other ENI in the same group (cluster communication).

![[SG-4.png]]

---

## 🏰 Bastion Host (Jumpbox)

> [!INFO] Definition
> A specializing EC2 instance in a **Public Subnet** used to provide secure management access to instances in **Private Subnets**.

*   **Architecture**:
    - **Inbound**: Connect via SSH/RDP from the internet.
    - **Outbound**: Jump to private instances using their private IPs.
*   **Security**: Should be heavily hardened and strictly limited to your source IP.
