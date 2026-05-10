---
tags:
  - aws/vpc
  - networking
  - endpoints
category: Networking
---

# 🔌 VPC Endpoints

> [!INFO] Definition
> **VPC Endpoints** allow resources inside a VPC to privately access AWS public services (like S3, DynamoDB, SNS) **without requiring an Internet Gateway, NAT Gateway, or public IP addresses**. Traffic stays entirely within the AWS network.
>
> There are two distinct types: **Gateway Endpoints** and **Interface Endpoints**.

---

## 🚪 Gateway Endpoints

Gateway Endpoints provide private access to **Amazon S3** and **Amazon DynamoDB** — the only two services supported.

### ⚙️ How It Works
1. You create a Gateway Endpoint and associate it with one or more subnets in your VPC.
2. AWS automatically adds a **Prefix List** entry to the Route Tables of the associated subnets.
3. Any traffic destined for S3 or DynamoDB leaves via the **Gateway Endpoint** rather than the Internet Gateway.

### 🏗️ Key Characteristics
* **Highly Available by Default**: A Gateway Endpoint is not placed inside a specific subnet or AZ; it is regional and automatically spans all AZs.
* **No Changes to Applications**: Because routing is handled at the Route Table level, applications are completely unaware of the endpoint — they still think they're communicating directly with S3/DynamoDB.
* **Regional Only**: Gateway Endpoints can only access services **in the same region**. Cross-region access is not supported.
* **Endpoint Policy**: An optional policy that restricts *which* resources the endpoint can access (e.g., limit access to specific S3 buckets only).
* **Not Accessible Outside the VPC**: Gateway Endpoints cannot be reached from on-premises networks or peered VPCs.

### 🎯 Use Cases

**Without Gateway Endpoint:**

![[GatewayEndpoints-1.png]]

* Public instances can reach S3 via their Public IPv4 → IGW → Public Internet → S3.
* Private instances must use a NAT Gateway to get a public IP → IGW → Public Internet → S3.
* **Problem**: Both paths require public internet routing. In a heavily regulated or air-gapped environment, this is not acceptable.

**With Gateway Endpoint:**

![[GatewayEndpoints-2.png]]

* Traffic from any instance (public or private) is routed via the VPC Router → Gateway Endpoint → S3 directly. No public IP required.
* **Use Case**: Lock S3 buckets to be accessible *only* via the Gateway Endpoint, preventing all public internet access to that bucket ("no leaky buckets").

> [!TIP] Exam PowerUP: Gateway Endpoints
> * Works with **prefix lists + route tables** — zero application changes required.
> * **HA by design** — regional, no per-AZ setup needed.
> * **Not accessible outside the VPC** (no on-prem or cross-VPC access).

---

## 🔗 Interface Endpoints

Interface Endpoints extend private access to almost **all other AWS public services** (and some S3, but not DynamoDB).

### ⚙️ How It Works
Interface Endpoints create an **Elastic Network Interface (ENI)** with a private IP address inside a specific subnet of your VPC. Traffic flows to the endpoint's private IP, which then routes it to the target AWS service via **AWS PrivateLink**.

* **PrivateLink**: The underlying technology that allows AWS services (or even third-party services) to be injected privately into your VPC.

### 🏗️ Key Characteristics
* **Not Highly Available by Default**: Each Interface Endpoint lives in one specific subnet (one AZ). For HA, you must deploy one Interface Endpoint **per AZ** used in the VPC.
* **Network Access**: Controlled via **Security Groups** (unlike Gateway Endpoints which use NACLs/Route Tables only).
* **Endpoint Policy**: Can restrict which actions or resources can be accessed via the endpoint.
* **Protocol**: Supports **TCP and IPv4 only**.

### 🌐 DNS Options

When you create an Interface Endpoint, you get multiple DNS names:

1. **Endpoint-Specific DNS** (e.g., `vpce-123-xyz.sns.us-east-1.vpce.amazonaws.com`): Resolves to the private IP of the Interface Endpoint. Applications must be explicitly configured to use this.
2. **Endpoint Regional DNS**: Resolves regardless of AZ — best for simplicity and high availability.
3. **Endpoint Zonal DNS**: Resolves to the specific interface in a specific AZ.

**PrivateDNS (Recommended)**:
* Associates a **Route 53 Private Hosted Zone** with the VPC.
* Overrides the *default* public DNS for the service (e.g., `sns.us-east-1.amazonaws.com`) — now resolves to the Interface Endpoint's **private IP** instead of a public IP.
* **No application changes required** — even unmodified apps that still use the default service DNS will automatically route through the Interface Endpoint.

### Traffic Flow Diagrams

**Without Interface Endpoints:**

![[InterfaceEndpoints-1.png]]

* Both public and private instances resolve `sns.us-east-1.amazonaws.com` to a public IP. Public instances can connect via IGW; private instances fail because they have no path to the public IP.

**With Interface Endpoints (no PrivateDNS):**

![[InterfaceEndpoints-2.png]]

* Apps using endpoint-specific DNS resolve to the endpoint's private IP and route through it. Apps still using default DNS continue routing via IGW.

**With Interface Endpoints + PrivateDNS:**

![[InterfaceEndpoints-3.png]]

* The default service DNS is overridden — *all* apps, even those not configured to use endpoint-specific DNS, now route through the Interface Endpoint automatically. This is the cleanest and most robust configuration.

---

### Gateway vs. Interface Endpoints

| Feature                    | Gateway Endpoint           | Interface Endpoint                     |
| :------------------------- | :------------------------- | :------------------------------------- |
| **Services**               | S3, DynamoDB only.         | Almost all AWS services.               |
| **Mechanism**              | Route Table + Prefix List. | ENI with Private IP (DNS-based).       |
| **High Availability**      | ✅ Regional, HA by default. | ❌ Per-AZ — must deploy one per AZ.     |
| **Application Changes**    | None required.             | None if PrivateDNS is enabled.         |
| **Security Control**       | Endpoint Policy.           | Security Groups + Endpoint Policy.     |
| **Accessible Outside VPC** | ❌ No.                      | Can work with PrivateLink (carefully). |
