---
tags:
  - aws/cloudhsm
  - security
  - encryption
category: Security & Encryption
---

# 🔐 AWS CloudHSM

> [!INFO] Definition
> **AWS CloudHSM** is a cloud-based hardware security module (HSM) that enables you to easily generate and use your own encryption keys on the AWS Cloud. It provides true "Single Tenant" dedicated hardware, unlike KMS which is a multi-tenant managed service.

---

## ✨ Key Features & Characteristics

* **Dedicated Hardware**: CloudHSM gives you exclusive access to a hardware appliance within the AWS cloud.
* **FIPS Compliance**: CloudHSM is **FIPS 140-2 Level 3** compliant (whereas KMS is broadly Level 2 overall, with some Level 3 components).
* **Industry Standard APIs**: Accessed via standard APIs like PKCS#11, Java Cryptography Extension (JCE), and Microsoft CryptoNG (CNG).
* **AWS Access**: AWS provisions the hardware, manages patches/maintenance, but they have **no access** to the secure area where your key material is held. (If you lose your credentials, your keys are gone permanently).
* **KMS Integration**: You can use CloudHSM as a **Custom Key Store** for AWS KMS.

## 🏗️ Architecture

![[CloudHSM-1.png]]

* **VPC Placement**: The HSM appliance is deployed into an AWS-managed VPC that you cannot see.
* **Network Interfaces**: It is injected into your own Customer VPC via an Elastic Network Interface (ENI).
* **High Availability**: By default, a single CloudHSM is **not Highly Available (HA)**.
  * To achieve HA, you must deploy multiple HSMs across different AZs and configure them as a **Cluster**.
  * The cluster automatically synchronises keys and policies between the appliances.
  * Your client instances must be configured to load-balance across all the ENIs.
* **Client Software**: To use the devices, the CloudHSM client software must be installed on your EC2 instances.

---

## 🎯 Use Cases & Exam PowerUP

Unlike KMS, CloudHSM has **no native AWS service integration** (except passing through KMS via custom key store). Use CloudHSM when you specifically need:

1. Offloading SSL/TLS processing for Web Servers.
2. Enabling **Transparent Data Encryption (TDE)** for Oracle databases.
3. Protecting the Private Keys for an **Issuing Certificate Authority (CA)**.
4. Compliance requirements explicitly demanding **FIPS 140-2 Level 3** single-tenant hardware.

> [!IMPORTANT] Exam Rule
> * **FIPS 140-2 Level 3** or **single-tenant hardware** → **CloudHSM**.
> * **Industry standard encryption APIs** (PKCS#11) → **CloudHSM**.
> * **Integration with AWS services** (S3, EBS, RDS) → **KMS**.
> * **FIPS 140-2 Level 2** → **KMS**.
