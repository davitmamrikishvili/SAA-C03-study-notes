---
tags:
  - aws/directory-service
  - identity
  - hybrid
category: Hybrid Connectivity
---

# 📂 AWS Directory Service

> [!INFO] Definition
> **AWS Directory Service** is a managed directory solution that runs **within a VPC** (Regional scope). It provides a centralized store of users, groups, devices, and policies — enabling single sign-on and centralized management across your AWS and on-premises environments.

## 🤔 What's a Directory?

Before diving into the AWS service, understand **why** directories exist:

- A **directory** is an inverted tree structure that stores identity-related objects: **users, groups, computers, file shares**.
- Multiple directories can be grouped into a **forest** (trust relationships between them).
- Commonly used in **Windows enterprise environments** — you join laptops, desktops, and servers to a directory.
- **Centralized authentication**: sign in once, access multiple devices/resources with the same credentials.
- Popular implementations: **Microsoft Active Directory Domain Services (AD DS)**, **Samba** (open-source).

> [!TIP] Why This Matters for the Exam
> If a question mentions "users need to sign in with their corporate credentials" or "centralized management of Windows machines in AWS," you're almost certainly looking at Directory Service.

---

## 🏗️ How AWS Directory Service Works

- It's a **private service** — it runs inside a specific **VPC** (Regional).
- AWS services that need a directory (e.g., **Amazon WorkSpaces**) must either:
  - Be **inside the same VPC**, or
  - Have **private connectivity** (VPC Peering, Transit Gateway, VPN) to that VPC.
- You choose from **three modes** depending on your relationship with existing on-premises directories.

> [!WARNING] Exam Pointer
> Directory Service is **Regional** (tied to a VPC). It is **NOT** a global service — you deploy it per region.

---

## 📋 The Three Modes

### 1. Simple AD

![[DirectoryService-1.png]]

- The **cheapest and simplest** option.
- Built on **Samba 4** (open-source), aiming for lightweight compatibility with Microsoft AD.
- **Two sizes:**
  - **Small** — up to 500 users
  - **Large** — up to 5,000 users
- Integrates with AWS services: Amazon Chime, Amazon Connect, RDS, AWS Management Console, EC2, and more.
- **Designed for isolation** — no integration with on-premises directories.

> [!DANGER] Critical Limitation
> Simple AD is **NOT** a full Microsoft AD implementation. If your application requires **actual Microsoft AD DS features** (e.g., trust relationships, group policy, schema extensions), you must use **AWS Managed Microsoft AD**.

---

### 2. AWS Managed Microsoft AD

![[DirectoryService-2.png]]

- **Full Microsoft AD DS** running in AWS, managed by AWS (patching, backups, multi-AZ deployment).
- Designed for **hybrid environments** — you have an existing on-premises AD and want presence in AWS.
- **Trust relationship** established with your on-premises directory over **private networking** (VPN or Direct Connect).
- **Trust direction**: The **primary directory lives in AWS** and trusts your on-premises directory.

> [!TIP] Resilience Benefit
> Because the primary directory is in AWS, if the VPN/Direct Connect fails, AWS services that depend on the directory **continue to function**. The on-premises trust may become unavailable, but the AWS-side directory remains operational.

- Directly supports applications that require **real Microsoft AD features** (e.g., SharePoint, SQL Server, .NET apps using Windows authentication).

---

### 3. AD Connector

![[DirectoryService-3.png]]

- A **proxy only** — no directory data is stored in AWS.
- Use case: You need to use an AWS service that requires a directory (e.g., Amazon WorkSpaces), but you **already have an on-premises directory** and don't want to maintain a second one.
- How it works:
  1. Establish **private connectivity** (VPN or Direct Connect) between your VPC and on-premises.
  2. Create an **AD Connector** that points to your on-premises domain controllers.
  3. AWS services use the connector to **proxy authentication requests** back on-premises.

> [!DANGER] Critical Failure Mode (Exam Favorite)
> If **private connectivity fails**, the AD Connector **stops working entirely** — and all AWS services relying on it will fail authentication. This is a key distinction from **Managed Microsoft AD**, which survives a connectivity loss because the directory is hosted in AWS.

---

## ⚖️ Picking the Right Mode

| Scenario                                                  | Mode to Choose               |
| :-------------------------------------------------------- | :--------------------------- |
| Simple requirements, directory in AWS only, no on-prem AD | **Simple AD**                |
| Need full Microsoft AD features OR trust with on-prem AD  | **AWS Managed Microsoft AD** |
| Already have on-prem AD, just need AWS services to use it | **AD Connector**             |

