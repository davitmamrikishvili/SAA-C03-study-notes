---
tags:
  - aws/foundations
  - architecture
category: Cloud Concepts
---

# 🌏 Global and Regional AWS Architecture

> [!INFO] Overview
> AWS infrastructure is built around **Regions** and **Availability Zones**, but some services operate at a **Global** level to provide entry points and optimization for worldwide traffic.

---

## 🏗️ Architectural Components

AWS services and products map onto major architectural components classified by their scope:

### 🌐 Global Components
These services operate across all AWS Regions and provide a unified entry point for users globally.
* **Service Location & Discovery**: Global DNS (Route 53) to direct users to the nearest endpoint.
* **Content Delivery & Optimization**: Amazon CloudFront (CDN) to cache data at Edge Locations for lower latency.
* **Global Health Checks & Failover**: Monitoring service health worldwide and routing traffic away from unhealthy regions.

### 📍 Regional Components
These services are locked to a specific AWS Region and are designed for high availability within that boundary.
* **Regional Entry Point**: Elastic Load Balancing (ELB) or API Gateway serving a specific region.
* **Regional Scaling & Resilience**: Auto Scaling groups and multi-AZ deployments within the region.
* **Application Services**: Compute (EC2), Databases (RDS/Aurora), and Storage (EBS) that live within a specific region.

---

## 🖼️ Visual Context

![[RegionalandGlobalArchitecture-1.png]]

![[RegionalandGlobalArchitecture-2.png]]

---

> [!TIP] Exam PowerUP: Scope Matters
> * **Global Services**: IAM, Route 53, CloudFront, WAF.
> * **Regional Services**: S3 (logic is regional, names are global), EC2, VPC, Lambda.
> * Always identify if a question asks for **global reach** (latency/CDN) or **regional resilience** (Multi-AZ).
