---
tags:
  - aws/ec2
  - compute
category: Compute
---

# 🧬 EC2 Instance Types & Lifecycle

## 🏷️ Decoding the Names
Example: <span style="color:rgb(255, 192, 0)">R</span><span style="color:rgb(255, 0, 0)">5</span><span style="color:rgb(0, 176, 240)">dn</span>.<span style="color:rgb(240, 75, 200)">8xlarge</span>
* **<span style="color:rgb(255, 192, 0)">Instance Family</span>**: (e.g., **R** for Memory)
* **<span style="color:rgb(255, 0, 0)">Instance Generation</span>**: (e.g., **5**)
* **<span style="color:rgb(0, 176, 240)">Additional Capabilities</span>**: (e.g., **d** for Disk, **n** for Network)
* **<span style="color:rgb(240, 75, 200)">Instance Size</span>**: (e.g., **8xlarge**)

### 📁 Instance Categories
| Category                  | Types            | Use Cases                                                          |
| ------------------------- | ---------------- | ------------------------------------------------------------------ |
| **General Purpose**       | T3, M5, A1       | Diverse workloads, balanced resources.                             |
| **Compute Optimized**     | C5, C6g          | Media processing, HPC, Scientific modeling, Gaming.                |
| **Memory Optimized**      | R5, X1, z1d      | High-performance databases, Real-time analytics, In-memory caches. |
| **Accelerated Computing** | P3, G4, F1, Inf1 | Machine Learning, GPU intensive, Genomics (FPGA).                  |
| **Storage Optimized**     | I3, D2, H1       | NoSQL, Data warehousing, HDFS, Kafka.                              |

---

## 🔄 Instance Lifecycle
* **Running**: Active and billing.
* **Stopped**: Data on EBS persists, but you aren't billed for compute (only for EBS storage).
* **Terminated**: Instance is deleted and cannot be restarted.

> [!TIP] Exam Nugget
> Stopping an instance and then starting it again changes its **Public IP** (unless it has an Elastic IP), but it keeps its **Private IP**.
