---
tags:
  - serverless
  - architecture
  - aws/sns
  - aws/sqs
category: Compute
---

# 🌩️ Serverless Architecture

> [!INFO] Definition
> Serverless is **not** about the absence of servers — it's about **not managing** them. The infrastructure is fully abstracted away.

### Key Principles
* **Small & Specialised**: Applications are composed of focused, single-purpose functions.
* **Stateless & Ephemeral**: Functions run in temporary environments with no data persisting between invocations.
* **Event-Driven**: Nothing runs unless an event triggers it — no idle compute.
* **FaaS-First**: Use **Function-as-a-Service** (e.g., Lambda) wherever possible for compute.
* **Pay-per-Use**: Billed for **actual execution time**, not for provisioned capacity sitting idle.

---

## 📢 SNS - Simple Notification Service

> [!INFO] Definition
> **Amazon SNS** is a fully managed, highly available **pub/sub messaging service**. It is a **Public AWS Service** accessed via a public endpoint.

### 🧩 Core Concepts
* **Topic**: The base entity of SNS. All permissions, configuration, and delivery settings are defined at the Topic level.
* **Publisher**: Any entity that **sends messages** to a Topic (e.g., CloudWatch Alarms, S3 Events, application code).
* **Subscriber**: Any entity that **receives messages** from a Topic (e.g., SQS, Lambda, HTTP/S endpoints, Email, SMS).
* **Message Size**: Payloads are limited to **≤ 256 KB**.

![[SNS-1.png]]

### 📡 Fan-Out Pattern
* A single message published to a Topic can be delivered to **multiple subscribers simultaneously** — this is the **fan-out** pattern.
* **Use case**: Decoupling microservices, distributed systems, and serverless applications (e.g., an S3 event fans out to both an SQS queue and a Lambda function).

### 🛡️ Features
* **Delivery Status**: Tracking for HTTP, Lambda, and SQS endpoints.
* **Delivery Retries**: Built-in reliable delivery with retry logic.
* **Resilience**: Highly Available, Scalable, and **Regionally Resilient**.
* **Encryption**: Supports **Server-Side Encryption (SSE)** for messages at rest.
* **Cross-Account Access**: Controlled via **SNS Topic Policies** (similar to S3 Bucket Policies).