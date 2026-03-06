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

---

## 🏥 Route 53 Health Checks

Health checks are independent objects in Route 53; they are created and configured separately from your DNS records but are utilized *by* them.

* **Global Fleet**: Checks are performed by a distributed fleet of health checkers operating globally.
* **Targets**: Not limited to AWS resources. You can check *anything* accessible over the public internet.
* **Intervals**: Standard checks occur every **30 seconds**. Fast checks occur every **10 seconds** (costs extra).
* **Protocols**: Supports TCP, HTTP/HTTPS, and HTTP/HTTPS with String Matching.
* **State**: Based on responses, an endpoint is marked as either <span style="color:rgb(0, 176, 80)">**Healthy**</span> or <span style="color:rgb(255, 0, 0)">**Unhealthy**</span>.
* **3 Types of Checks**:
	1.  **Endpoint**: Monitor a specific IP or Domain.
	2.  **CloudWatch Alarm**: Monitor an AWS service's health via an alarm status.
	3.  **Calculated (Checks of Checks)**: Combine multiple health checks (e.g., "healthy if 3 out of 5 endpoint checks pass").

![[Route53HealthChecks.png]]

---

## 🚦 Routing Policies

### Simple Routing
* **Logic**: One record mapped to a name. It returns the value(s) in the record.
* **Use Case**: When you want to route all requests toward a single service (e.g., an individual web server).
* **Limitation**: It is the **ONLY** routing policy that does **not** support associated Health Checks.

![[Route53SimpleRouting-1.png]]

### Failover Routing
* **Logic**: Implements an Active/Passive architecture using a primary and secondary record.
* **Health Check Dependency**: The primary record *must* have an associated health check. If <span style="color:rgb(0, 176, 80)">healthy</span>, Route 53 returns the primary value.
* **Failover**: If the primary is <span style="color:rgb(255, 0, 0)">unhealthy</span>, Route 53 returns the secondary value.
* **Use Case**: Routing to an "out-of-band" static maintenance page (e.g., hosted on S3) when your main application goes down.

![[Route53FailoverRouting-1.png]]

### Multi-Value Answer Routing
* **Logic**: You create multiple records with the same name. Route 53 returns a list of up to 8 healthy records.
* **Availability**: Improves availability (the client can try another IP if the first fails), but it is **NOT** a replacement for true Load Balancing (due to client-side DNS caching).
* **Health Checks**: Every record should have an associated health check so only healthy IPs are returned to the client.

![[Route53MultiValueRouting-1.png]]

### Weighted Routing
* **Logic**: You assign a numeric weight to each record (e.g., 90 vs. 10). Traffic is routed proportionally based on the total weight.
* **Use Case**: A simple form of load balancing, or conducting **A/B Testing / Canary Deployments** (sending a small percentage of traffic to a new software version).

![[Route53WeightedRouting-1.png]]

### Latency-Based Routing
* **Logic**: AWS maintains a latency database mapping user locations to AWS Regions. Route 53 returns the record corresponding to the region offering the **lowest network latency** for that specific user.
* **Use Case**: Optimizing for raw performance and the best user experience.
* **Caveat**: The AWS database is not real-time and cannot account for sudden, localized ISP routing issues.

![[Route53LatencyBasedRouting-1.png]]

### Geolocation Routing
* **Logic**: Resolution decisions are based strictly on the geographical location of the requesting user.
* **Scope**: Records are tagged with locations (US State, Country, Continent, or a "Default" catch-all).
* **Unlike Latency**: It does *not* necessarily return the "closest" record; it returns the exact record you specified for that location.
* **Use Case**: Enforcing regional compliance restrictions, delivering language-specific content, or enforcing licensing laws.

![[Route53GeolocationRouting-1.png]]

### Geoproximity Routing
* **Logic**: Routes traffic based on the geographic location of your resources and your users.
* **Traffic Flow**: This policy **must** be created using **Route 53 Traffic Flow** (the visual editor).
* **The "Bias" Feature**: You can optionally configure a "Bias" (+ or -) to expand or shrink the geographic footprint of a specific region, effectively shifting traffic from one region to another.
* **Use Case**: When you want more granular geographic control than simple Geolocation and need the ability to mathematically shift regional boundaries.

![[Route53GeoProximityRouting-1.png]]

---

### 📊 Routing Policy Comparison

> [!TIP] Exam PowerUP: Choosing the Right Policy
> Memorize this table. Questions will almost always present a scenario where one of these is the precise answer.

| Policy           | Keyword / Core Use Case                                         | Health Checks?   |
| :--------------- | :-------------------------------------------------------------- | :--------------- |
| **Simple**       | Single service / basic routing.                                 | ❌ NO             |
| **Failover**     | Active/Passive, **"Maintenance Page."**                         | ✅ YES (Required) |
| **Multi-Value**  | Simple HA, return multiple IPs. Not an ELB.                     | ✅ YES            |
| **Weighted**     | **A/B Testing**, Canary releases, proportional splits.          | ✅ YES            |
| **Latency**      | Best **performance**, lowest millisecond response.              | ✅ YES            |
| **Geolocation**  | **Compliance**, language, strict regional borders.              | ✅ YES            |
| **Geoproximity** | **"Traffic Flow"**, shrinking/expanding regions via **"Bias."** | ✅ YES            |

---

## 🤝 Interoperability (Registrar vs. Hosting)

Route 53 performs two distinct functions. You can use Route 53 for **both** simultaneously, or use **either one independently** (e.g., register on GoDaddy, host on Route 53).

1. **Domain Registrar**: The service that registers your domain name with the Top-Level Domain (TLD) registry and manages your registration fee.
2. **Domain Hosting**: The service that holds your DNS database (Zone File) and answers DNS queries over the internet.

### 🔄 The Complete AWS Registration Workflow
When you purchase a new domain directly through Route 53, the following sequence occurs automatically:

1. **Payment** *(Registrar)*: Route 53 accepts your domain registration fee.
2. **Allocation** *(Hosting)*: Route 53 allocates **4 Name Servers (NS)** specifically for your new domain.
3. **Zone Creation** *(Hosting)*: Route 53 creates the DNS Zone File on those 4 Name Servers. (The Domain Hosting setup is now complete).
4. **Communication** *(Registrar)*: Route 53 communicates with the external TLD registry (e.g., the root `.com` registry).
5. **Delegation** *(Registrar)*: Route 53 sets the NS records at the TLD registry to point to the 4 AWS Name Servers allocated in Step 2, gluing the components together.

> [!TIP] Exam PowerUP: Separation of Concerns
> Exams often test your understanding that these are two separate jobs. If a domain is registered externally, you can still use Route 53 for **Domain Hosting** by manually performing Step 5: taking the 4 Name Servers AWS gives you and pasting them into the external registrar's dashboard.