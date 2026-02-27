---
tags:
  - aws/ec2
  - storage
category: Storage
---

# Instance Store

## ⚡ Instance Store Volumes

> [!INFO] Definition
> **Instance Store** provides temporary block-level storage physically attached to the **EC2 Host** computer.

*   **Ephemeral**: Data persists only during the lifetime of its associated instance on that specific host.
    *   **Survives**: OS Reboots.
    *   **Lost on**: Instance Stop, Hibernate, Termination, or Hardware Failure.
*   **Host Physics**: Because the disk is physically inside the same box as the CPU/RAM, it provides the **highest possible performance** (IOPS/Throughput) in AWS due to zero network latency.
*   **Provisioning**: Must be defined at **launch time**. You cannot attach an instance store volume to an instance that is already running.

---

## 🏗️ Choosing: EBS vs. Instance Store

| Feature | EBS | Instance Store |
| :--- | :---: | :---: |
| **Persistence** | ✅ Permanent | ❌ Temporary |
| **Resilience** | ✅ Replicated by AWS | ❌ No replication |
| **Performance** | ⚠️ High (Network) | ✅ Extreme (Local) |
| **Detachable** | ✅ Yes | ❌ No |
| **Cost** | ⚠️ Billed separately | ✅ Included in Instance price |

---

## 🛡️ RAID 0 for Performance Scaling

> [!TIP] The Concept
> When a single EBS or Instance Store volume isn't fast enough, you can use **RAID 0 (Striping)** at the OS level.

*   **How it works**: Data is stripped across multiple volumes, combining their IOPS and Throughput.
*   **The Bandwidth Cap**: Your total performance is ultimately capped by the **Max EBS Bandwidth of the Instance Type**.
*   **Risk**: RAID 0 has **no redundancy**. If one volume in the set fails, all data on the RAID drive is lost.
