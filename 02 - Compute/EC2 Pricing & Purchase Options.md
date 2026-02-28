---
tags:
  - aws/ec2
  - pricing
category: Compute
---

# 💰 EC2 Pricing & Purchase Options

## 💰 Core Purchase Options

### 🟢 On-Demand
* **Most Flexible**: The default choice. No long-term commitment and no upfront cost.
* **Isolation**: Multiple customer instances run on shared hardware, but are logically isolated.
* **Billing**: Pay-per-second while running. (EBS still bills when instance is stopped).
* **Best For**: Short-term workloads, unpredictable traffic, or testing new applications.
* **Constraint**: No capacity reservation (you might get an "Insufficient Capacity" error during peak times).

![[EC2PurchaseOptions-1.png]]

### 🟡 Spot Instances
* **Deepest Discount**: Up to **90% cheaper** than On-Demand by using spare AWS capacity.
* **Variable Pricing**: The "Spot Price" fluctuates based on demand/supply.
* **Interruption**: If AWS needs the capacity back or the spot price exceeds your max bid, you get a **2-minute warning** before the instance is terminated.
* **Best For**: Stateless, fault-tolerant, or flexible workloads (Batch jobs, Data analysis, CI/CD).
* **Rule of Thumb**: Never use Spot for stateful workloads that cannot tolerate interruption.

![[EC2PurchaseOptions-2.png]]

### 🔵 Reserved Instances (RI)
* **Commitment**: 1 or 3-year term for significant discounts.
* **Scope vs. Capacity**:
    * **Zonal RI**: Locked to a specific AZ. **Reserves capacity** but only applies discount in that AZ.
    * **Regional RI**: Applies to any AZ in the region. **No capacity reservation**, but more flexible for scaling.
* **Payment Options**:
    * **All Upfront**: Highest discount.
    * **Partial Upfront**: Balanced.
    * **No Upfront**: Lowest discount for agreeing to the term.
* **Standard vs. Convertible**: Standard RIs have higher discounts but cannot change instance families; Convertibles allow family changes.

![[EC2PurchaseOptions-3.png]]

### 🛡️ Dedicated Hosts
* **Full Hardware Control**: A physical server is allocated entirely to you.
* **Billing**: You pay for the **Host**, not the instances running on it.
* **Licensing (BYOL)**: Essential for software licensed by **physical sockets/cores** (e.g., Oracle, Windows Server).
* **Host Affinity**: You can ensure specific instances always run on the same physical hardware.

![[EC2PurchaseOptions-4.png]]

### 🏢 Dedicated Instances
* **Hardware Isolation**: Instances run on hardware that is physically separated from other AWS customers.
* **Difference from Hosts**: You don't get control over the physical server attributes (sockets/cores). You just guarantee your neighbors are *you*.
* **Cost**: Includes an additional hourly fee per region where they are used.

![[EC2PurchaseOptions-5.png]]

---

## ⏱️ Advanced Purchase Options

### ⏱️ Scheduled Reserved Instances
* **Best For**: Workloads that run on a predictable schedule but do not run 24/7.
* **The Commitment**: A 1-year term with a minimum requirement of 1,200 hours per year.
* **Example**: A financial reporting job that must run every Monday morning for 4 hours. You pre-book that specific window for the year.
* **Limitations**: Does not support all instance types or regions.

### 🏟️ Capacity Reservations & Priority
AWS manages capacity using a specific priority hierarchy to ensure high-availability and reliability.

#### 📊 The Capacity Hierarchy
1. **Reservations (Zonal RIs / On-Demand Reservations)**: AWS fulfills these first because they are pre-booked.
2. **On-Demand**: Fulfilled based on remaining capacity.
3. **Spot**: Fulfilled using whatever is left over. (This is why Spot is the first to be terminated).

#### 📍 Zonal vs. Regional Reservations
* **Regional RI**: Provides a **billing discount** for valid instances in any AZ within the region. It is flexible but **does not reserve capacity**. In a major fault, you still might face "Insufficient Capacity" errors.
* **Zonal RI**: Provides both a **billing discount and a capacity reservation** in one specific Availability Zone.
* **On-Demand Capacity Reservation**: You can book capacity in a specific AZ **without a term commitment**. You pay the full On-Demand price for the reserved capacity, whether you use it or not.

---

## 💳 EC2 Savings Plans
The modern alternative to Reserved Instances, offering significantly more flexibility.

* **Commitment**: An hourly spend commitment (e.g., $10/hour) for a 1 or 3-year term.
* **Types**:
    * **Compute Savings Plan**: The most flexible. Automatically applies to EC2, Fargate, and Lambda regardless of instance family, region, size, or OS. (Up to 66% discount).
    * **EC2 Instance Savings Plan**: specific to an instance family within a region. Offers higher discounts (up to 72%) but less flexibility.
* **Consumption**: Usage consumes the commitment at the reduced "Savings Plan rate." Any usage beyond the commitment is billed at the standard On-Demand rate.
