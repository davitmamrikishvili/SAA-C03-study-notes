---
tags:
  - aws/elb
  - networking
  - high-availability
category: Networking
---

# ⚖️ ELB - Elastic Load Balancer

> [!INFO] Definition
> **AWS Elastic Load Balancing (ELB)** automatically distributes incoming application traffic across multiple targets, such as EC2 instances, containers, IP addresses, and Lambda functions. It handles the varying load of your application traffic in a single Availability Zone or across multiple Availability Zones.

---

## 🏎️ ELB Evolution & Types

AWS provides three main types of load balancers, categorized into **v1 (Classic)** and **v2 (Next-Gen)**. You should always prefer v2 for modern applications.

| Type                  | Version | Use Case                                     | Protocols                    |
| :-------------------- | :------ | :------------------------------------------- | :--------------------------- |
| **ALB (Application)** | v2      | Advanced Layer 7 routing (Path/Host-based).  | HTTP, HTTPS, gRPC, WebSocket |
| **NLB (Network)**     | v2      | Ultra-high performance, Layer 4, Static IPs. | TCP, TLS, UDP                |
| **CLB (Classic)**     | v1      | Legacy applications. (Avoid/Migrate).        | HTTP, HTTPS, TCP, SSL        |

---

## 🏗️ Architecture & How It Works

Understanding how ELB is physically deployed in your VPC is critical for the exam.

![[ELB-1.png]]

### 1. Nodes and DNS
* **Elasticity**: What appears as a single load balancer object is actually a collection of **Load Balancer Nodes**.
* **Deployment**: You must select at least **two Availability Zones** (one subnet per AZ). AWS then provisions one or more nodes into those subnets.
* **DNS Resolution**: ELB provides a **single DNS Name** (A Record). This DNS name resolves to the IP addresses of all currently active nodes.
* **HA**: Traffic is distributed equally across all nodes, ensuring high availability even if an entire AZ fails.

### 2. Networking & Subnet Requirements
* **IP Requirements**: For ELB to scale, each subnet it uses needs **8 or more free IP addresses**.
* **Addressing**: You must choose between **IPv4 only** or **Dual Stack** (IPv4 & IPv6).
* **Subnet Size**: AWS recommends a **/27 or larger** subnet to allow for effective node scaling.
> [!IMPORTANT] Exam Nugget
> If you see a question where /27 is not an option, **/28** is usually the minimum supported size.

---

## 🌐 Internet-Facing vs. Internal

The choice between Internet-facing and Internal determines the network accessibility of the load balancer nodes.

| Configuration       | Node IPs             | Primary Use                                                        |
| :------------------ | :------------------- | :----------------------------------------------------------------- |
| **Internet-Facing** | Public & Private IPs | Accepting traffic from the public internet.                        |
| **Internal**        | Private IPs only     | Distributing traffic between application tiers (e.g., Web to App). |

> [!CAUTION] IMPORTANT FOR EXAM
> An **Internet-facing** load balancer **can** connect to backend instances that are in **private subnets**. The instances do not need to be public; only the ELB nodes need to be public to receive traffic.

---

## 🛠️ Listener Configuration

A **Listener** is a process that checks for connection requests using a protocol and port that you configure.
* **Forwarding**: You define **Rules** that determine how the load balancer routes requests to its registered targets.
* **Tiering**: Internal load balancers are frequently used to separate different application tiers, allowing each tier (e.g., Web, App, DB) to scale independently and maintain tight security boundaries.

---

> [!TIP] Exam PowerUP: ELB Architecture Nuggets
> * **Abstraction**: Users connect to the ELB DNS; they are completely abstracted away from the physical EC2 instances.
> * **Scaling**: ELB scales its nodes automatically based on incoming traffic volume.
> * **Classic (v1)**: Avoid CLB unless the app requires features not available in v2 (extremely rare). CLB is limited to **1 SSL certificate** and lacks Layer 7 intelligence.

---

## 🔄 Cross-Zone Load Balancing

Historically, Load Balancer nodes were restricted to distributing traffic only to targets within their **own Availability Zone**.

* **The Problem**: If AZA had 4 instances and AZB had 1, each LB node would still split traffic 50/50 between the AZs. This resulted in the instance in AZB being heavily overloaded.
* **The Solution**: **Cross-Zone Load Balancing** allows each Load Balancer node to distribute traffic equally across **all registered instances** in all enabled Availability Zones.
* **Default Behavior**:
    * **Application Load Balancer (ALB)**: Always enabled by default (no charge for inter-AZ data transfer).
    * **Network Load Balancer (NLB)**: Disabled by default. If enabled, you may be charged for data transfer between AZs.

![[ELB-3.png]]
