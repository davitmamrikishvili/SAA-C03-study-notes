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

* **Static NAT**: What the Internet Gateway (IGW) does to map a Private IP to a Public IP 1-to-1.
* **IP Masquerading (Hide NAT)**: Hiding an entire CIDR block behind a single Public IP. This provides **outgoing-only** internet access.

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
* **Public Subnet**: Both NAT Gateways and NAT Instances **MUST** reside in a public subnet.
* **Route Tables**: Private subnets must have a route (e.g., `0.0.0.0/0`) pointing to the NAT device.
* **Regional Resilience**: Deploy one NAT Gateway **in each AZ** for full regional high availability.
* **NAT Instance Technicality**: You **MUST** disable the **Source/Destination Check** on the NAT instance's configuration for it to route traffic properly.

---

## 🌐 IPv6 & Egress-Only Internet Gateway (EIGW)

> [!TIP] No NAT for IPv6
> NAT Gateways and NAT Instances **do not work with IPv6**. All IPv6 addresses in AWS are Global Unicast Addresses (publicly routable by default), so NAT (which hides private IPs) is not applicable.

This creates a challenge: a standard **Internet Gateway** allows **both inbound and outbound** traffic for IPv6. If you want to give IPv6 instances **outbound-only** internet access (equivalent to what a NAT Gateway does for IPv4), you need an **Egress-Only Internet Gateway**.

### ⚙️ How It Works
* **Outbound Only for IPv6**: Allows instances in a VPC to initiate outbound connections to the internet over IPv6, but prevents the internet from initiating inbound connections to those instances.
* **Highly Available by Default**: Automatically spans all AZs in the region and scales as required — no management overhead.

![[Egress-Only Internet Gateway.png]]

### 🗺️ Route Table Configuration
After attaching an Egress-Only IGW, you must update the route tables in the relevant subnets:
* Add a default IPv6 route: **`::/0`** with the **`eigw-id`** as the target.

### Connectivity Summary
| Traffic Type                      | Gateway to Use                          |
| :-------------------------------- | :-------------------------------------- |
| IPv4 outbound (private instances) | NAT Gateway / NAT Instance              |
| IPv6 bi-directional               | Internet Gateway (IGW)                  |
| IPv6 outbound-only                | **Egress-Only Internet Gateway (EIGW)** |

