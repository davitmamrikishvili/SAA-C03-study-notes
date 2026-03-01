---
tags:
  - aws/ec2
  - compute
  - scaling
category: Compute
---

# ⚖️ EC2 Scaling (Vertical vs. Horizontal)

Scaling is the process of adjusting the computing capacity available to your application based on demand.

---

## 🏗️ Vertical Scaling (Scaling UP/DOWN)
> [!INFO] Definition
> Changing the **size** of an existing instance (e.g., moving from a `t3.micro` to a `t3.large`).

### ✅ Benefits
* **Simplicity**: No modification required to the application code.
* **Compatibility**: Works for all applications, including legacy monoliths that aren't designed for distributed systems.
* **Database Friendly**: Often the easiest way to scale traditional relational databases.

### ❌ Drawbacks & Limitations
* **Downtime**: Resizing an instance requires a **Stop & Start** (reboot), resulting in service disruption.
* **Hardware Cap**: You are limited by the maximum size of the instance family available in that region.
* **Price Premium**: Extremely large instances often carry a cost premium compared to several smaller ones.

---

## 🚀 Horizontal Scaling (Scaling OUT/IN)
> [!INFO] Definition
> Adding or removing **more instances** of the same size to handle the workload.

### ✅ Benefits
* **High Availability**: No disruption during scaling operations. If one instance fails, others continue to serve traffic.
* **No Limits**: You can theoretically scale out to hundreds or thousands of instances.
* **Cost Efficiency**: No "large instance" premium. You can be very granular with how much capacity you add.
* **Managed Services**: Works perfectly with **Auto Scaling Groups (ASG)** and **Elastic Load Balancers (ELB)**.

### ❌ Drawbacks & Complexity
* **Application Design**: Requires the application to be "Stateless" or use **Off-Host Sessions** (stored in ElastiCache or DynamoDB).
* **Load Balancers**: Requires an ELB to distribute traffic across the pool of instances.
* **Complexity**: More complex to monitor and manage multiple distributed resources.

---

> [!TIP] Exam PowerUP: The Selection Logic
> * If the question mentions **"No downtime"** or **"Unlimited Scale"** $\to$ Choose **Horizontal Scaling**.
> * If the question mentions scaling a **"Monolith"** with minimal effort $\to$ Choose **Vertical Scaling**.