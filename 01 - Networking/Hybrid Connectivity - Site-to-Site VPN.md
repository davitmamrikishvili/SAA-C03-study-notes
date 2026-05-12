---
tags:
  - aws/vpn
  - aws/bgp
  - networking
  - hybrid
category: Networking
---

# 🔌 Hybrid Connectivity - AWS Site-to-Site VPN

> [!INFO] Definition
> **AWS Site-to-Site VPN** creates a logical, encrypted tunnel between your VPC and an on-premises network over the public internet, using the **IPSec** protocol. It can be provisioned in under an hour and is often used as an initial or backup hybrid connectivity solution.

---

## 📡 BGP - Border Gateway Protocol

Site-to-Site VPN supports two modes: static and dynamic. The dynamic mode relies on **BGP**, so understanding it is essential.

* **Autonomous System (AS)**: BGP's fundamental unit. An AS is a network (or collection of networks) controlled by a single entity. BGP treats each AS as a black box — it only cares about the paths *between* them.
* **ASN (Autonomous System Number)**: A unique number assigned by IANA to each AS. Range: `0–65535`. Private ASNs: `64512–65534`.
* **Protocol**: Operates over **TCP port 179**. Reliable and distributed by design.
* **Path-Vector Protocol**: BGP exchanges the best **path** (called the **ASPATH**) to a destination between peers — not link speed or condition.
* **AS Path Prepending**: A technique to artificially inflate the ASPATH to make a route appear less preferable, effectively influencing which path is chosen.
* **Peering**: **Not automatic** — BGP peering relationships must be manually configured.
* **iBGP**: Internal BGP — routing *within* an AS.
* **eBGP**: External BGP — routing *between* ASes.

![[BGP101-1.png]]

---

## 🏗️ Building Blocks of Site-to-Site VPN

| Component                         | Description                                                                                                                                                                              |
| :-------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **VPC**                           | The AWS network being connected.                                                                                                                                                         |
| **Virtual Private Gateway (VGW)** | A logical gateway object in AWS, attached to one VPC. It is the AWS-side anchor for VPN connections. Physically, it has multiple endpoints in different AZs, making it **HA by design**. |
| **Customer Gateway (CGW)**        | Represents the on-premises router. Has two meanings: the **logical config object** in AWS, and the **physical router device** on-premises. You must specify its public IP.               |
| **VPN Connection**                | Created between a VGW and a CGW. It stores the configuration and creates tunnels.                                                                                                        |

---

## ⚙️ How a VPN Connection is Created

1. Gather: the VPC IP range, the on-premises network IP range, and the customer router's public IP.
2. Create a **VGW** and attach it to the VPC. Add it as a route table target.
3. Create a **CGW** in AWS, pointing to the customer router's public IP.
4. Create a **VPN Connection** linking the VGW and CGW.
5. AWS creates **2 VPN tunnels** — one from each of the VGW's physical endpoints to the customer router. As long as one tunnel is active, the networks are connected.

![[AWS-Site-2-SiteVPN-1.png]]

> [!WARNING] Partial HA (Default Setup)
> With a single customer router, there is a **single point of failure** on the on-premises side. Even though the VGW is HA with two endpoints, both tunnels connect back to *one physical device*. If that device fails, connectivity is lost.

---

## 🛡️ Achieving Full High Availability

To eliminate the single point of failure, you need a **second customer router**, preferably at a different physical location with a separate internet connection.

* Create a second CGW in AWS for the new router.
* Create a **second VPN connection** from the same VGW to the new CGW.
* This creates 4 tunnels total (2 per VPN connection), each connecting to a different customer router.
* Result: **Fully HA** — no single device failure can break connectivity.

![[AWS-Site-2-SiteVPN-2.png]]

---

## ⚖️ Static vs. Dynamic VPN

![[AWS-Site-2-SiteVPN-3.png]]

| Feature          | Static VPN                         | Dynamic VPN (BGP)                                              |
| :--------------- | :--------------------------------- | :------------------------------------------------------------- |
| **Routing**      | Manually configured static routes. | BGP automatically exchanges network routes.                    |
| **Failover**     | No automatic multi-link failover.  | Can use multiple links simultaneously.                         |
| **Route Tables** | Routes added manually.             | Supports **Route Propagation** — routes learned automatically. |
| **Requirement**  | None — works with any router.      | Customer router must support **BGP**.                          |
| **Flexibility**  | Simple but rigid.                  | Dynamic, self-healing, preferred for production.               |

* **Route Propagation**: When enabled on a VPC route table, any networks learned via active dynamic VPN connections are automatically added as routes — zero manual configuration needed.

---

## 📊 Considerations & Exam Facts

> [!IMPORTANT] Exam PowerUP: Site-to-Site VPN Key Facts
> * **Speed Cap**: ~**1.25 Gbps** limit on a VGW — a hard ceiling that cannot be exceeded.
> * **Latency**: **Inconsistent** — traffic traverses the public internet.
> * **Cost**: AWS hourly charge + per-GB data transfer out + any data cap on the customer's internet connection.
> * **Setup Time**: Can be provisioned in **under an hour** (all software configuration).
> * **IPSec**: Widely supported. Dynamic VPN additionally requires BGP on the customer router.
> * **Backup for Direct Connect**: Site-to-Site VPN is commonly used as a **cost-effective failover** for AWS Direct Connect (DX).
> * **Complementary to Direct Connect**: Can be used *alongside* DX for added resilience.
