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
* **Same Account**: You can associate a Private Hosted Zone with one or more VPCs using the AWS Console, CLI, or API.
* **Cross-Account**: You can associate a zone with VPCs in **different AWS accounts**, but this is only possible via the **CLI or API** (not the console). This requires an authorization step from the zone owner and an association step from the VPC owner.

![[PrivateHostedZones-1.png]]

---

## 🌓 Split-Horizon (Split-View) DNS

**Split-Horizon DNS** is a technique where you maintain two hosted zones (one Public, one Private) with the **exact same domain name** (e.g., `corp.example.com`).

* **Internal Users**: When a user inside an associated VPC queries the name, they receive the **Private** record (e.g., an internal IP like `10.0.1.5`).
* **External Users**: When a user on the public internet queries the name, they receive the **Public** record (e.g., a public Elastic IP).
* **Use Case**: This allows you to have a single URL for your application while presenting different content (like an Intranet) to employees while customers see the public site.

![[SplitView.png]]

> [!TIP] Exam PowerUP: Authoritative Source
> Remember that a Hosted Zone is authoritative for a domain because it is referenced via **Delegation** using NS records. For Private zones, this delegation is handled internally by the **Route 53 Resolver** within your VPC.

---

## 🏷️ Route 53: CNAME vs. Alias Records

Choosing between a CNAME and an Alias record is a recurring exam topic. While both map one name to another, the **Alias record** is a proprietary AWS feature with critical advantages.

### ⛓️ The CNAME Limitation (Apex Domain Problem)
A **CNAME (Canonical Name)** maps one name to another name. Standard DNS rules prohibit a CNAME from existing at the **Zone Apex** (the root or "naked" domain like `catagram.io`).

* **Standard Record**: An `A` record maps a name to an IP.
* **The Conflict**: Standard DNS forbids a CNAME from co-existing with any other records (like SOA or NS records) for the same name. Since the apex *must* have SOA/NS, a CNAME is **invalid** at the apex.
* **AWS Context**: If you want your root domain (`catagram.io`) to point to a Load Balancer (ELB), a standard CNAME is not an option.

### 🎯 The Alias Solution
An **Alias record** is a virtual pointer that maps a name directly to an **AWS Resource** (ELB, S3, CloudFront, etc.). It resolves the "Apex Problem" internally.

* **Apex Support**: Alias records **can** be used for the **Apex (Naked Domain)**.
* **Efficiency**: Resolution happens in one step. Route 53 returns the IP of the resource directly, avoiding the extra DNS lookup "hop" required by CNAMEs.
* **Cost Efficiency**: DNS queries to Alias records that point to AWS resources are **FREE**.
* **Dynamic**: If the underlying IP of an ELB changes, Route 53 automatically updates the pointer.
* **Type Matching**: You specify the record type (**A** or **AAAA**) that the resource responds to. For example, if pointing to an **Elastic Load Balancer**, you create an **A Type Alias Record**.

> [!IMPORTANT] Exam PowerUP: The Selection Logic
> - **Always** select **Alias** over CNAME for any supported AWS service.
> - **Apex Domain Requirement**: You **must** use an **Alias**; CNAME is forbidden centrally.
> - **Cost**: Alias is **free** for AWS resources; CNAME attracts query charges.
> - **Requirement**: You can only use Alias if Route 53 is your DNS provider (it's AWS specific).
> - **Matching**: Ensure the Alias type (A/AAAA) matches what the resource supports.

| Feature               | CNAME                | Alias                        |
| :-------------------- | :------------------- | :--------------------------- |
| **Zone Apex Support** | ❌ NO                 | ✅ YES                        |
| **Mapping Target**    | NAME to NAME         | NAME to AWS Resource         |
| **Query Charge**      | Standard Cost        | **FREE** (for AWS resources) |
| **Performance**       | Standard (Extra Hop) | Native (One Hop)             |