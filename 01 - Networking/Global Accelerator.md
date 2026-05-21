---
tags:
  - aws/global-accelerator
  - networking
  - performance
category: Networking
---

# 🚀 AWS Global Accelerator

> [!INFO] Definition
> **AWS Global Accelerator** is a networking service that improves the availability and performance of your applications by routing traffic through AWS's private global backbone network, rather than the unpredictable public internet.

---

## 🏗️ How It Works

### 📡 Anycast IP Addresses
Global Accelerator provisions **two static Anycast IP addresses** as the entry point for your application.

* **Unicast (Normal IP)**: Points to one specific device. Two devices sharing the same IP causes conflicts.
* **Anycast**: Multiple devices can share the same IP address simultaneously. Routers automatically direct traffic to the **topologically closest** device advertising that IP.

This means all of your users worldwide send traffic to the same two IP addresses, and they are automatically routed to the nearest Global Accelerator **Edge Location** — without any DNS complexity.

### 🌐 The Traffic Path
1. A user sends traffic to one of the two Anycast IP addresses.
2. Due to Anycast routing, the traffic enters the **nearest Global Accelerator Edge Location** via the public internet.
3. From that edge location, the data transits entirely across the **AWS private global backbone network** — a massively optimised, high-throughput, low-latency private network.
4. Traffic is delivered to one or more of your application **endpoints** (EC2 instances, ALBs, NLBs, or Elastic IPs in any AWS region).

![[GlobalAccelerator-2.png]]

> [!TIP] The Key Insight
> The public internet is used **only for the very first hop** (user → nearest edge). All remaining transit uses AWS's private backbone, avoiding congestion, packet loss, and unpredictable latency of the public internet.

---

## ⚖️ Global Accelerator vs. CloudFront

These two services are commonly confused in the exam. Both leverage AWS's global edge network, but they solve fundamentally different problems.

| Feature              | CloudFront                              | Global Accelerator                                     |
| :------------------- | :-------------------------------------- | :----------------------------------------------------- |
| **Primary Function** | Content **caching** at edge locations.  | Network **performance optimisation** via AWS backbone. |
| **Protocol Support** | HTTP/HTTPS **only**.                    | **Any TCP or UDP** application.                        |
| **Caching**          | ✅ Yes — the core feature.               | ❌ No — it does not cache anything.                     |
| **Use Cases**        | Websites, APIs, media, static assets.   | Gaming, IoT, VoIP, real-time apps, global failover.    |
| **Client-Facing IP** | Dynamic, DNS-based (CloudFront domain). | **Static Anycast IPs** (consistent, IP-whitelistable). |

> [!IMPORTANT] Exam PowerUP: CloudFront or Global Accelerator?
> * **"Caching"** or **"HTTP/HTTPS content delivery"** → **CloudFront**.
> * **"TCP/UDP"**, **"non-HTTP"**, **"static IP required"**, or **"global performance for any application"** → **Global Accelerator**.
> * Global Accelerator has **no understanding of HTTP** — it is purely a Layer 3/4 network product.

