---
tags:
  - aws/vpc
  - networking
  - monitoring
category: Networking
---

# 📊 VPC Flow Logs

> [!INFO] Definition
> **VPC Flow Logs** capture **metadata** about the IP traffic flowing to and from network interfaces within your VPC. They are a critical tool for network monitoring, security analysis, and troubleshooting.

> [!WARNING] Important Limitation
> Flow Logs capture **packet metadata only** (source IP, destination IP, ports, protocol, action). They do **NOT** capture packet contents. If you need to inspect the actual data payload, you must install a **packet sniffer** directly on the EC2 instance.

---

## 🏗️ Attachment Levels

Flow Logs can be attached at three different levels of granularity. Logs are inherited downward — attaching at the VPC level automatically monitors all interfaces within it.

![[VPCFlowLogs-1.png]]

| Level                       | Scope                                                       |
| :-------------------------- | :---------------------------------------------------------- |
| **VPC**                     | Captures traffic for **all ENIs** within the entire VPC.    |
| **Subnet**                  | Captures traffic for **all ENIs** within a specific subnet. |
| **ENI (Network Interface)** | Captures traffic for **one specific ENI** only.             |

---

## ⚙️ Key Characteristics

* **Not Real-Time**: There is a delay between traffic flowing through the monitored interface and the logs appearing in the destination. Flow Logs are not suitable for real-time alerting.
* **Log Filtering**: You can configure logs to capture **ACCEPTED**, **REJECTED**, or **ALL** traffic. This is useful for reducing storage costs and focusing on relevant data.
* **Destinations**:
    * **Amazon CloudWatch Logs** — for real-time monitoring, metric filters, and alarms.
    * **Amazon S3** — for long-term storage, bulk analysis, and cost efficiency.
    * **Amazon Athena** — can be used on top of S3 to run SQL queries against the flow log data.

---

## 📋 Log Record Structure

Each Flow Log entry is a row with a defined set of fields.

![[VPCFlowLogs-2.png]]

Key fields include: `srcaddr`, `dstaddr`, `srcport`, `dstport`, `protocol` and `action` (ACCEPT/REJECT).
