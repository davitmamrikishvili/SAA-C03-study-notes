---
tags:
  - aws/ec2
  - networking
  - vpc
category: Compute
---

# 🌐 EC2 Networking & DNS Architecture

> [!INFO] Elastic Network Interface (ENI)
> An ENI represents a virtual network card. It is the logical component in a VPC that connects your instances to the network.

## 🧬 ENI Characteristics
* **Primary ENI (eth0)**: Automatically created and attached when an instance is launched. You **cannot detach** the primary ENI.
* **Secondary ENIs**: Can be manually created and attached/detached while the instance is running.
    * **AZ Boundary**: An ENI must be in the same Availability Zone (AZ) as the instance, but it can be in a **different subnet**.
* **Components of an ENI**:
    * **MAC Address**: A unique hardware identifier visible to the Guest OS.
    * **Primary Private IPv4**: Fixed for the life of the ENI.
    * **Secondary Private IPv4s**: Multiple IPs can be assigned to a single ENI.
    * **Public IPv4**: 0 or 1 Public IP (assigned by AWS or via Elastic IP).
    * **Elastic IP (EIP)**: A static, public IPv4 address that can be moved between ENIs/instances.
    * **IPv6 Addresses**: 0 or more.
    * **Security Groups**: SGs are attached to the **ENI**, not the instance.

![[EC2networking-2.png]]

---

## 🛡️ Source/Destination Check
> [!WARNING] Important for NAT/Firewalls
> By default, every ENI performs a "Source/Destination Check." It only allows traffic if the instance is the **source** or the **intended destination** of the packet.

* **When to Disable**: You must **disable** this check for any instance acting as a "middleman" (Router, NAT Instance, or Firewall) so it can forward packets that aren't addressed to it.

---

## 🎨 IP Addressing & DNS Behavior

### 🧩 Public vs. Private IPs
* **OS Visibility**: The Guest Operating System **never sees its Public IP**. It only knows about its Private IP.
* **Dynamic Nature**: Public IPv4 addresses are **dynamic**. If you STOP and START an instance, it receives a **new** public IP. (Note: Elastic IPs are static and solve this).

### 📍 DNS Resolution Logic
* **Public DNS Name**: AWS provides a public DNS name (e.g., `ec2-54...`).
* **Internal Resolution**: If queried from within the same VPC, the public DNS resolves to the **Private IP** (saving bandwidth and lowering latency).
* **External Resolution**: If queried from outside the VPC (the internet), it resolves to the **Public IP**.

---

## 🚀 Exam PowerUP: ENI Use Cases

1. **Licensing Management**: Some legacy software is "locked" to a specific MAC address. By using a **Secondary ENI**, you can detach it from an old instance and move it to a new one, keeping the MAC address (and the license) intact.
2. **Management Networks**: Use multiple ENIs to create "Multi-homed" instances. For example, `eth0` for public data traffic and `eth1` for a secure, private management/logging subnet.
3. **The Elastic IP Trap**: If an instance has a standard public IPv4 and you assign an **Elastic IP**, you lose the original IP forever. If you later remove the EIP, the instance will get a *brand new* dynamic public IP, not the original one.
