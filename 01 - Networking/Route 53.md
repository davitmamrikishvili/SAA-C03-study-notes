---
tags:
  - aws/route53
  - networking
  - dns
category: Networking & Content Delivery
---

# 🌐 AWS Route 53

> [!INFO] Definition
> Amazon Route 53 is a highly available and scalable Cloud Domain Name System (DNS) web service. It is designed to give developers and businesses an extremely reliable and cost-effective way to route end users to Internet applications.

---

## 🏗️ Public Hosted Zones

A **Public Hosted Zone** is a container that holds information about how you want to route traffic on the internet for a specific domain (e.g., `animals4life.org`) and its subdomains.

### 🧩 Core Characteristics
* **Authoritative DNS**: Route 53 acts as the authoritative source for your DNS records.
* **Global Resilience**: Your DNS records are hosted across multiple redundant DNS servers globally.
* **Architecture**: When you create a Public Hosted Zone, Route 53 automatically allocates **4 Public Nameservers**.
* **Integration**: To use Route 53 for a domain, you must update the domain's **Name Server (NS)** records at your registrar to point to these 4 Route 53 nameservers.
* **External Domains**: You can use Route 53 to host DNS for domains registered with third-party registrars (e.g., GoDaddy, Namecheap).

### 📝 Resource Records (RR)
Inside a hosted zone, you create Resource Records, which are the actual data units DNS uses:
* **A / AAAA**: Mapping names to IP addresses.
* **CNAME / Alias**: Mapping names to other names or AWS resources.
* **MX**: Mail exchange records.
* **TXT**: Text records (often used for verification).

### 💰 Pricing Model
* **Monthly Fee**: There is a fixed monthly cost per hosted zone.
* **Query Fee**: A small charge per million queries made against the zone.

![[PublicHostedZones-1.png]]

---

## 🏘️ Private Hosted Zones

A **Private Hosted Zone** is a DNS container that is only accessible and resolvable from within specified **Virtual Private Clouds (VPCs)**. Unlike Public zones, these records are never exposed to the public internet.

### 🔗 VPC Association
*   **Same Account**: You can associate a Private Hosted Zone with one or more VPCs using the AWS Console, CLI, or API.
*   **Cross-Account**: You can associate a zone with VPCs in **different AWS accounts**, but this is only possible via the **CLI or API** (not the console). This requires an authorization step from the zone owner and an association step from the VPC owner.

![[PrivateHostedZones-1.png]]

---

## 🌓 Split-Horizon (Split-View) DNS

**Split-Horizon DNS** is a technique where you maintain two hosted zones (one Public, one Private) with the **exact same domain name** (e.g., `corp.example.com`).

*   **Internal Users**: When a user inside an associated VPC queries the name, they receive the **Private** record (e.g., an internal IP like `10.0.1.5`).
*   **External Users**: When a user on the public internet queries the name, they receive the **Public** record (e.g., a public Elastic IP).
*   **Use Case**: This allows you to have a single URL for your application while presenting different content (like an Intranet) to employees while customers see the public site.

![[SplitView.png]]

> [!TIP] Exam PowerUP: Authoritative Source
> Remember that a Hosted Zone is authoritative for a domain because it is referenced via **Delegation** using NS records. For Private zones, this delegation is handled internally by the **Route 53 Resolver** within your VPC.