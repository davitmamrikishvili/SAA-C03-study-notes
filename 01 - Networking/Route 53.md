---
tags:
  - aws/route53
  - networking
  - dns
category: Networking & Content Delivery
---

# 🌐 AWS Route 53

Amazon Route 53 is a highly available and scalable Cloud Domain Name System (DNS) web service. It is designed to give developers and businesses an extremely reliable and cost-effective way to route end users to Internet applications.

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

## 🏘️ Private Hosted Zones (Quick Intro)
* **Scope**: Only accessible from within specified **VPCs**.
* **Usage**: Used for internal service discovery and private networking environments where you don't want your DNS records exposed to the public internet.

> [!TIP] Exam PowerUP: The Source of Truth
> A Hosted Zone is authoritative for a domain because it is referenced via **Delegation** using NS records. This tells the global DNS system: "For anything related to `myapp.com`, go ask these Route 53 servers."
