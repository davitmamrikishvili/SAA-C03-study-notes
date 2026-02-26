---
tags:
  - aws/vpc
  - networking
category: Networking & Content Delivery
---

# VPC NAT & Connectivity

## 🛠️ Network Address Translation (NAT)

> [!INFO] Definition
> NAT is the process of allowing a resource with a **Private IP** to communicate with external networks (like the Internet) while remaining private.

*   **Static NAT**: What the Internet Gateway (IGW) does to map a Private IP to a Public IP 1-to-1.
*   **IP Masquerading (Hide NAT)**: Hiding an entire CIDR block behind a single Public IP. This provides **outgoing-only** internet access.

---

## ⚡ NAT Gateway vs. NAT Instance

AWS provides two main ways to handle NAT:

| Feature | NAT Gateway (Managed) | NAT Instance (EC2) |
| :--- | :--- | :--- |
| **Availability** | Highly available within an AZ. Scales automatically. | Must be managed manually. Use scripts for failover. |
| **Bandwidth** | Scales up to 45 Gbps. | Depends on the instance type. |
| **Maintenance** | Managed by AWS (No patching). | Managed by YOU (Patching, OS updates). |
| **Security Groups** | **NOT SUPPORTED**. Uses NACLs only. | Supported (It's just an EC2). |
| **Cost** | Fixed hourly rate + Data processing charges. | Instance hourly rate + Data transfer charges. |

### 🛠️ Configuration Details
*   **Public Subnet**: Both NAT Gateways and NAT Instances **MUST** reside in a public subnet.
*   **Route Tables**: Private subnets must have a route (e.g., `0.0.0.0/0`) pointing to the NAT device.
*   **Regional Resilience**: Deploy one NAT Gateway **in each AZ** for full regional high availability.
*   **NAT Instance Technicality**: You **MUST** disable the **Source/Destination Check** on the NAT instance's configuration for it to route traffic properly.

---

## 🌐 IPv6 & Connectivity

> [!WARNING] No NAT for IPv6
> NAT Gateways and NAT Instances **do not work with IPv6**. All IPv6 addresses in AWS are Global Unicast Addresses (publicly routable).

### Connectivity Types
1.  **Bi-directional**: Route `::/0` through an **Internet Gateway (IGW)**.
2.  **Outbound-Only**: Route `::/0` through an **Egress-only Internet Gateway (EIGW)**. This is the IPv6 equivalent of a NAT Gateway (allows outgoing traffic but blocks initiating inbound connections).
