---
tags:
  - aws/direct-connect
  - networking
  - hybrid
category: Hybrid Connectivity
---

# 🔌 AWS Direct Connect (DX)

> [!INFO] Definition
> **AWS Direct Connect** is a dedicated physical network connection between your on-premises network and AWS. Unlike Site-to-Site VPN, traffic does **not** traverse the public internet — it uses a private, direct fiber link via an AWS DX Location.

> [!CAUTION] No HA and No Encryption by default
> A standard DX connection provides **no built-in redundancy** and **no native encryption**. Both must be explicitly designed for.

---

## 🏗️ Physical Architecture

### 📡 Port Speeds & Standards
| Speed       | Physical Standard                    |
| :---------- | :----------------------------------- |
| **1 Gbps**  | 1000BASE-LX (Single-mode fiber)      |
| **10 Gbps** | 10GBASE-LR (Single-mode fiber)       |
| **40 Gbps** | Link Aggregation (4 × 10 Gbps ports) |

### 🔗 The Connection Components
Think of a DX connection conceptually as a **single fiber optic cable** from an AWS-managed DX port all the way to your network:

1. AWS allocates a **DX Port** at a physical **DX Location** when you order.
2. You establish a **cross-connect** — a physical cable from the AWS DX rack to your (or your partner's) router, also inside the DX Location.
3. Your router needs to support **VLANs and BGP**.
4. You (or a Partner) then extend the connection from the DX Location back to your **business premises or data center**.

> [!TIP] Small Business / No DX Presence?
> If you don't have equipment at a DX Location, connect to an **AWS Partner router** at the location and let the partner manage the last-mile connectivity to your premises.

![[DirectConnect-1.png]]

---

## 🌐 Virtual Interfaces (VIFs)

A single physical DX connection can carry multiple **Virtual Interfaces (VIFs)**. Each VIF is a **VLAN + BGP session** between your router and the AWS DX router, logically isolated from each other.

### Types of VIFs

| VIF Type        | Purpose                                                                | Connection Target                                                   |
| :-------------- | :--------------------------------------------------------------------- | :------------------------------------------------------------------ |
| **Public VIF**  | Access to **AWS Public Zone services** (S3, SQS, DynamoDB, SNS, etc.). | AWS Public Zone. NOT the public internet.                           |
| **Private VIF** | Private network connectivity to a **single VPC** via its VGW.          | One VPC per Private VIF. No limit on number of Private VIFs per DX. |

---

## ⚖️ Considerations vs. Site-to-Site VPN

| Factor                 | Direct Connect (DX)                            | Site-to-Site VPN                        |
| :--------------------- | :--------------------------------------------- | :-------------------------------------- |
| **Provisioning Time**  | Weeks to months (physical cabling).            | Under an hour (software only).          |
| **Max Bandwidth**      | Up to **40 Gbps** (aggregated).                | ~**1.25 Gbps** (VGW cap).               |
| **Latency**            | **Consistently low** — dedicated private link. | **Variable** — public internet transit. |
| **Encryption**         | ❌ None natively.                               | ✅ IPSec encrypted by default.           |
| **Internet Bandwidth** | Doesn't use your business internet circuit.    | Uses your existing internet connection. |
| **HA**                 | ❌ None by default — single point of failure.   | ✅ Dual tunnels via VGW by design.       |
| **Cost**               | Higher (port hours + data transfer).           | Lower (hourly + per-GB).                |

> [!TIP] Recommended Pattern
> Provision a **Site-to-Site VPN first** for immediate connectivity, then order DX and replace (or keep VPN as a resilient backup) once DX is provisioned.

---

## 🔒 Adding Encryption to DX (The Workaround)

Natively, DX has no encryption. To achieve encrypted traffic with DX's performance benefits, you combine both services:

![[DirectConnect-2.png]]

1. Create a **Public VIF** over your DX connection.
2. Since a **VGW** has endpoints in AWS's public zone (with public IPs), you can establish an **IPSec Site-to-Site VPN using the Public VIF as transit** — instead of using the public internet.
3. Result: You get the **low latency and consistent performance of DX** combined with the **IPSec encryption of a VPN**.

> [!IMPORTANT] Exam PowerUP: DX + VPN = Encrypted Private Connectivity
> If an exam scenario requires **both DX performance and encryption**, the answer is to run a **Site-to-Site VPN over a Public VIF** on top of a DX connection.
