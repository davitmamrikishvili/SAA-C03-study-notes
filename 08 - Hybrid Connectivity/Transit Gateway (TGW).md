---
tags:
  - aws/transit-gateway
  - networking
  - hybrid
category: Hybrid Connectivity
---

# 🔀 AWS Transit Gateway (TGW)

> [!INFO] Definition
> **AWS Transit Gateway** is a highly available and scalable network transit hub that connects your Amazon VPCs and on-premises networks together using a hub-and-spoke topology.

> [!IMPORTANT] Core Benefit: Transitive Routing
> Unlike standard VPC Peering or Site-to-Site VPNs (which do **not** support transitive routing), Transit Gateway acts as a central router. It simplifies architecture by eliminating the need for complex, full-mesh network designs.

---

## 🏗️ Architecture & Topology

To use a Transit Gateway, you create **Attachments**. An attachment is the way TGW interfaces with other network objects.

### Valid Attachments
* **VPC** (acts like an inter-VPC router)
* **Site-to-Site VPN** (TGW becomes the AWS-side termination point instead of a VGW)
* **Direct Connect Gateway**
* **Other Transit Gateways** (TGW Peering)

### Before vs. After TGW

**Without Transit Gateway (Full Mesh):**
* If you have 5 VPCs that all need to talk to each other and back to on-premises, you have to configure VPC peering connections and VPN connections between *every single pair*.
* **Problems**: High administrative overhead, scales poorly, complex route tables.
![[TransitGateway-1.png]]

**With Transit Gateway (Hub and Spoke):**
* Every VPC and on-premises network (via VPN or DX) connects **only** to the Transit Gateway.
* The TGW routes traffic between all attached networks.
![[TransitGateway-2.png]]

---

## ⚙️ Configuration Details

* **VPC Attachments**: When you attach a VPC, you specify **one subnet in each AZ** that you want the Transit Gateway to use. Once attached, it operates as a highly available router within those AZs.
* **Route Tables**: TGWs come with a default route table that automatically routes traffic between attachments. You can also create multiple custom route tables to implement complex routing topologies (e.g., isolating production VPCs from development VPCs).

---

## 🌐 Scaling & Global Networks

Transit Gateway is designed to be the backbone of massive, global enterprise networks.

* **Cross-Account Sharing**: You can share a Transit Gateway across multiple AWS accounts using **AWS RAM (Resource Access Manager)**. This is perfect for large organizations with multi-account strategies (AWS Organizations).
* **Cross-Region Peering**: You can peer a Transit Gateway in one AWS region with another Transit Gateway in a different region (even across different accounts). Traffic between peered TGWs remains entirely on the AWS global private backbone.
