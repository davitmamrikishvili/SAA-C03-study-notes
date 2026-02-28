---
tags:
  - aws/ec2
  - networking
  - security
category: Compute
---

# 🧬 EC2 Instance Metadata (IMDS)

> [!INFO] Definition
> **Instance Metadata Service (IMDS)** is a service provided by AWS to every EC2 instance. It contains data about the instance that can be used to configure or manage it once it is running.

## 📍 Accessing Metadata
Accessible only from **within the instance** via a special Link-Local address (IPv4 and IPv6).

* **IPv4 Address**: `http://169.254.169.254/latest/meta-data/`
* **Example Command**: `curl http://169.254.169.254/latest/meta-data/instance-id`

---

## 🔎 What Data is Available?
* **Networking**: Public IP, Private IP, VPC ID, Subnet ID, Security Groups.
* **Environment**: AZ, Region, AMI ID, Instance Type.
* **Authentication**: **IAM Role Security Credentials** (TEMPORARY credentials used by applications running on the instance).
* **Launch Data**: Instance Hostname, Profile ARN.
* **User Data**: Configuration scripts provided at launch are accessible at `http://169.254.169.254/latest/user-data/`.

---

## 🛡️ Security & Protocol Versions (v1 vs v2)

> [!WARNING] Important for the Exam
> Instance Metadata is **NOT** authenticated or encrypted. Use IMDSv2 for better protection against SSRF (Server-Side Request Forgery) attacks.

### 🧩 IMDSv1
* **Method**: Uses a simple **GET** request.
* **Vulnerability**: Susceptible to SSRF attacks because any process that can trigger a GET can steal the metadata.

### 🛡️ IMDSv2 (Session-Oriented)
* **Method**: Requires a **Token** via a `PUT` request before you can perform a `GET`.
* **Security**: The token is session-based and significantly more secure against SSRF.

---

### 🚀 Exam PowerUP: EC2 Identity & IMDS
* **IAM Roles**: When you attach an IAM Role to an instance, the **temporary credentials** (Access Key, Secret Key, Token) are provided to the instance via the `meta-data/iam/security-credentials/` endpoint.
* **IMDS vs. User Data**: User Data is used for **Configuration** (running a script), whereas Metadata is for **Discovery** (finding out about the environment).
