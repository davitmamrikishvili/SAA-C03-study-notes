---
tags:
  - aws/sqs
  - messaging
  - integration
category: Application Integration
---

# 📬 SQS - Simple Queue Service

> [!INFO] Definition
> **Amazon SQS** is a fully managed, public, highly available message queuing service. It enables the decoupling of application components so that they can run and fail independently, increasing the overall resilience of the system.

---

## 🏗️ Queue Types

SQS offers two distinct queue types:

| Feature | Standard Queue | FIFO Queue |
| :--- | :--- | :--- |
| **Ordering** | Best-effort ordering. | Strict **First-In-First-Out** ordering. |
| **Delivery** | **At-least-once** (messages can be delivered more than once). | **Exactly-once** processing. |
| **Performance** | Nearly unlimited throughput. | Up to **3,000 msg/s** with batching, or **300 msg/s** without. |

---

## 💰 Billing & Limits

* **Billed per Request**: 1 request = 1 to 10 messages, up to **256 KB total** payload.
* **Message Size**: Individual messages can be up to **256 KB**. For larger payloads, store the data externally (e.g., in S3) and include a reference link in the message.
* **Retention**: Messages can remain in the queue for up to **14 days**.
* **Encryption**: Supports encryption **at rest** (KMS) and **in-transit**.

---

## 🔄 Polling: Short vs. Long

How a consumer retrieves messages from the queue is a key architectural choice.

* **Short Polling (Immediate)**: The consumer makes a request and receives 0 or more messages immediately. Can result in many empty responses and higher costs per request.
* **Long Polling (<span style="color:rgb(240, 75, 200)">waitTimeSeconds</span>)**: The consumer specifies a wait time (up to **20 seconds**). The request stays open until a message arrives or the timer expires. This **reduces the number of API calls** and is the **recommended** approach.

---

## 👻 VisibilityTimeout & Message Lifecycle

1. A **producer** sends a message to the queue.
2. A **consumer** polls the queue and receives the message. The message is **not deleted** — it becomes temporarily **hidden** for a period called the <span style="color:rgb(240, 75, 200)">VisibilityTimeout</span>.
3. The consumer processes the workload the message represents.
4. **Success**: The consumer explicitly **deletes** the message from the queue.
5. **Failure**: If the consumer crashes or doesn't delete the message before the VisibilityTimeout expires, the message **reappears** in the queue and becomes available for another consumer to process.

* **Dead-Letter Queues (DLQ)**: Messages that fail processing repeatedly (exceed a configured receive count) are moved to a separate DLQ for debugging and analysis.

---

## 🔗 Integration Patterns

SQS is a cornerstone of event-driven and decoupled architectures on AWS:

* **ASG Scaling**: Auto Scaling Groups can scale worker instances based on the <span style="color:rgb(240, 75, 200)">ApproximateNumberOfMessagesVisible</span> metric (queue depth). See: [[02 - Compute/Auto Scaling Groups|Auto Scaling Groups]].
* **Lambda Invocation**: Lambda functions can be triggered directly by messages arriving in an SQS queue.

### 📣 Fan-Out Pattern (SNS + SQS)

> [!IMPORTANT] Exam PowerUP: The Fan-Out Design
> A common exam scenario: An object is uploaded to an S3 bucket, and the requirement is to trigger **multiple independent processing jobs** from that single event.
> * **The Problem**: S3 event notifications can only send one event to one destination per rule.
> * **The Solution**: Configure S3 to send the event to a single **SNS Topic**. Then subscribe **multiple SQS queues** to that SNS topic. Each queue receives a copy of the message, allowing multiple independent consumers to process the same event in parallel.
> * **Keywords**: "Decouple" + "Multiple consumers from single event" → **SNS Fan-Out to SQS**.
