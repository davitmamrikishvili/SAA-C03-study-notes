---
tags:
  - aws/kinesis
  - streaming
  - integration
category: Application Integration
---

# 🌊 Kinesis - Data Streaming

> [!INFO] Definition
> **Amazon Kinesis** is a scalable, fully managed streaming service designed to collect, process, and analyze real-time data at any scale. It is a **public service** and highly available by design.

---

## 🏗️ Kinesis Data Streams: Architecture

### 📡 The Stream
* **Producers** (applications, IoT devices, servers) continuously send data into a **Kinesis Stream**.
* Streams can scale from low throughput to near-infinite data rates.
* **Data Retention**: Streams store data in a **24-hour rolling window** by default. This can be extended up to **365 days** (7 days at standard pricing).
* **Multiple Consumers**: Unlike SQS (where messages are consumed and deleted), multiple consumers can independently read from the same stream's rolling window simultaneously.

### 🧱 Shards: The Unit of Scale
* A Kinesis Stream is composed of one or more **Shards**.
* Each shard provides a fixed unit of capacity:
    * **Ingestion**: 1 MB/s (or 1,000 records/s).
    * **Consumption**: 2 MB/s.
* **Scaling**: More shards = more throughput = higher cost.

### 📦 Kinesis Data Record
* The individual unit of data stored in a stream.
* Maximum size: **1 MB**.

---

## 🔥 Kinesis Data Firehose

**Kinesis Data Firehose** is a fully managed delivery service that connects to a Kinesis Stream and automatically loads the streaming data into AWS destinations like:
* **Amazon S3** (most common)
* **Amazon Redshift** (via S3 intermediary)
* **Amazon Elasticsearch Service**
* **Third-party services** (Splunk, Datadog, etc.)

It handles batching, compression, and encryption automatically — ideal for moving streaming data into persistent storage at scale.

![[Kinesis.png]]

---

## ⚖️ SQS vs. Kinesis: Decision Guide

This is a critical exam distinction. Both handle data flow, but they serve fundamentally different purposes.

| Feature              | SQS                                                    | Kinesis                                                  |
| :------------------- | :----------------------------------------------------- | :------------------------------------------------------- |
| **Primary Use**      | Decoupling & async communication.                      | Large-scale data **ingestion** & streaming.              |
| **Consumer Model**   | 1 production group → 1 consumption group.              | Multiple independent consumers from the same stream.     |
| **Data Persistence** | No persistence. Messages are deleted after processing. | **Rolling window** (24h default, up to 365 days).        |
| **Ideal For**        | Worker pools, task queues, decoupling tiers.           | Real-time analytics, monitoring, IoT, app click streams. |

> [!IMPORTANT] Exam PowerUP: SQS or Kinesis?
> * **"Streaming"**, **"ingestion"**, **"real-time analytics"**, or **"large number of devices"** → **Kinesis**.
> * **"Decoupling"**, **"worker pool"**, **"async communication"** → **SQS**.
