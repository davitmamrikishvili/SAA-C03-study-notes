---
tags:
  - aws/shield
  - aws/waf
  - security
  - networking
category: Security & Encryption
---

# 🛡️ AWS Shield & WAF

> [!INFO] Definition
> **AWS Shield** and **AWS WAF (Web Application Firewall)** represent the two primary mechanisms for protecting AWS perimeter infrastructure against Distributed Denial of Service (DDoS) attacks and malicious web exploits.

---

## 🛡️ AWS Shield (Layer 3 & 4 Protection)

AWS Shield provides managed protection against **DDoS attacks** targeting the network and transport layers (OSI Layers 3 & 4), such as SYN floods or UDP reflection attacks.

It comes in two tiers:

### 1. Shield Standard
* **Cost**: Free (automatically enabled for all AWS customers).
* **Coverage**: Protects against common, frequently occurring Layer 3/4 network/transport DDoS attacks.
* **Integrations**: Natively protects **Route 53** and **CloudFront** endpoints.

### 2. Shield Advanced
* **Cost**: $3,000 / month flat fee.
* **Expanded Coverage**: Extends enhanced protection to **EC2 instances**, **Elastic Load Balancers (ELB)**, and **Global Accelerator**.
* **DDoS Response Team (DRT)**: Provides 24/7 access to the AWS DRT who can assist with real-time attack mitigation and custom rule creation.
* **Financial Protection**: Offers "cost protection" — if your AWS infrastructure automatically scales up to absorb a DDoS attack, AWS will refund the cost spikes caused by the malicious traffic.

---

## 🧱 Web Application Firewall - WAF (Layer 7 Protection)

AWS WAF is a **Layer 7 (Application Layer)** firewall designed to protect web applications or APIs against common web exploits that could affect availability, compromise security, or consume excessive resources.

### Key Features
* Protects against complex attacks such as **SQL Injection (SQLi)** and **Cross-Site Scripting (XSS)**.
* Can filter traffic based on **IP addresses, geo-locations, headers, or rate limits** (e.g., blocking an IP making >1000 requests per minute).
* Configured using **Web Access Control Lists (Web ACLs)**. Rules are added to a Web ACL, evaluated in order, and then traffic is either Allowed, Blocked, or Counted.

### Native Integrations
WAF can be attached only to specific services:
1. **Application Load Balancer (ALB)** (Regional)
2. **API Gateway** (Regional)
3. **CloudFront** (Global edge distribution)
4. **AppSync** (Regional)

---

## 🏗️ Architecture Example: Defense in Depth

Combining CloudFront, Shield, and WAF provides a comprehensive "Defense in Depth" perimeter configuration.

![[ShieldAndWAF.png]]

1. **The Edge**: Traffic hits **Route 53** and **CloudFront** first. These are automatically protected from massive volume attacks by **Shield Standard** (L3/L4).
2. **Global WAF**: A **Web ACL** is attached to the CloudFront distribution. This filters out malicious Application Layer (L7) traffic (like SQL injections) at the global edge locations *before* it reaches your backend.
3. **The Origin**: CloudFront forwards clean traffic to the origin, which in this case is an **ALB**.
4. **Regional Protection**: If you subscribe to **Shield Advanced**, the ALB is granted enhanced L3/L4 protection. You can also attach a *second* WAF Web ACL directly to the ALB to provide a secondary layer of regional L7 filtering.
5. **VPC Entry**: By the time traffic enters your VPC, it has already been scrubbed at Layers 3, 4, and 7.

> [!TIP] Exam PowerUP
> * **SQL Injection / XSS / Geo-blocking / Layer 7** → **WAF**.
> * **DDoS Protection / Layer 3 & 4 / SYN Floods** → **Shield**.
> * **Shield Advanced benefits** → $3k/month, protects ELB/EC2/Global Accelerator, includes 24/7 support & automatic scaling cost refunds.
> * WAF **cannot** be deployed directly onto an EC2 instance — it must be attached to an ALB, API Gateway, or CloudFront.
