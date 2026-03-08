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

---

## 🏢 Load Balancer Consolidation & SNI

Historically, the **Classic Load Balancer (CLB)** lacked modern features like **SNI (Server Name Indication)**. This meant every unique HTTPS application required its own dedicated load balancer, leading to high architectural overhead and costs.

* **What is SNI?**: A TLS extension that allows a server to host multiple separate SSL/TLS certificates on a single IP address. The client indicates which hostname it is trying to connect to during the initial handshake.
* **ALB Consolidation**: **Application Load Balancers (ALB)** support SNI and advanced routing rules. You can host multiple domains (e.g., `app1.com` and `app2.com`) on a single ALB using different SSL certificates and route them to different **Target Groups**.

![[ALBvsNLB-1.png]]

---

## 🍕 ALB (Application Load Balancer) - L7

ALB operates at **Layer 7 (Application Layer)** and is designed for the modern web.

### 🧠 Intelligence & Features
* **Protocol Support**: Specifically handles **HTTP**, **HTTPS**, **gRPC**, and **WebSockets**.
* **Content-Aware**: It can "inspect" traffic to make routing decisions based on:
    * **Host Headers** (e.g., `orders.example.com` vs `images.example.com`)
    * **Path Patterns** (e.g., `/api/*` vs `/static/*`)
    * **HTTP Headers, Cookies, and Query Strings**.
* **Health Checks**: Evaluates the actual application health (e.g., checking for a `200 OK` response from a specific page).

### 🔒 Security & SSL
* **SSL Termination**: The ALB **terminates** the SSL/TLS connection from the client. It decrypts the traffic, inspects it, and creates a *new* connection to the backend.
* **No Unbroken SSL**: You cannot pass an encrypted TCP stream directly to an instance without the ALB decrypting it first.


![[ALBvsNLB-2.png]]

---

## 🏎️ NLB (Network Load Balancer) - L4

NLB operates at **Layer 4 (Transport Layer)** and is built for extreme performance and non-HTTP protocols.

### 🚀 Performance & Connectivity
* **Speed**: Capable of handling **millions of requests per second** with ultra-low latency (approx. 25% of an ALB).
* **Protocols**: Ideal for **TCP**, **UDP**, **TLS**, and specialized apps (SMTP, Gaming servers, Financial data feeds).
* **Static IPs**: Each AZ used by the NLB gets a **Static IP address**. You can also assign **Elastic IPs**. This is critical for applications where the client needs a whitelist-able firewall entry.

### 🔗 PrivateLink Foundation
> [!IMPORTANT] The PrivateLink Requirement
> The **Network Load Balancer** is the only ELB type that can be used as a front-end for **AWS PrivateLink** (VPC Endpoint Services). This allows you to expose a service in your VPC to thousands of other VPCs (even across different AWS accounts) completely over the AWS private network, without using IGWs or public IPs.

### 🔒 Security: Pass-Through
* **Unbroken Encryption**: NLB can be configured with **TCP Listeners** to "pass-through" traffic without decrypting it. This allows for **unbroken end-to-end encryption** between the client and your instance.

> [!TIP] Exam PowerUP: Choosing between ALB & NLB
> * If a question mentions **Static IP** or **Whitelisting** -> **NLB**.
> * If you need **unbroken encryption** -> **NLB**.
> * If you need fastest performance -> **NLB**.
> * Protocols other than **HTTP** or **HTTPS** -> **NLB**.
> * If you need to share a service privately (**PrivateLink**) -> **NLB**.
> * Otherwise -> **ALB**.

---

## 🔗 Related: Elastic Scaling

ELBs are most powerful when paired with **Auto Scaling Groups (ASGs)** and **Launch Templates** to deliver fully elastic, self-healing architectures.

> [!INFO] Deep Dive
> For Launch Templates vs. Launch Configurations, ASG scaling policies, health checks, and ELB integration patterns, see the dedicated note: [[02 - Compute/Auto Scaling Groups|Auto Scaling Groups]].