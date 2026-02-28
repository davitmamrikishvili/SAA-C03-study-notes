---
tags:
  - aws/databases
  - concept
category: Databases
---

# 🗄️ Database Fundamentals

Before choosing an AWS database service, it is critical to understand the trade-offs between different consistency models and the limits of distributed systems.

---

## 📐 The CAP Theorem

> [!INFO] Definition
> The CAP theorem states that it is impossible for a distributed data store to simultaneously provide more than two out of the following three guarantees:

| Characteristic          | Description                                                                                                                    |
| :---------------------- | :----------------------------------------------------------------------------------------------------------------------------- |
| **Consistency**         | Every read receives the most recent write or an error.                                                                         |
| **Availability**        | Every request receives a (non-error) response, but without the guarantee that it contains the most recent write.               |
| **Partition Tolerance** | The system continues to operate despite an arbitrary number of messages being dropped or delayed by the network between nodes. |

> [!WARNING] The Choice
> In a distributed system, network partitions are a reality (P is mandatory). Therefore, you must choose between **Consistency (CP)** or **Availability (AP)**.

---

## 🏗️ Transaction Models: ACID vs. BASE

### 🛡️ ACID (Focus: Consistency)
Traditional Relational Databases (RDS) follow the ACID model to ensure data integrity.

* **Atomic**: All or nothing. If one part of a transaction fails, the entire transaction fails.
* **Consistent**: Transactions move the database from one valid state to another, respecting all rules/constraints.
* **Isolated**: Concurrent transactions do not interfere with each other; they execute as if they were serial.
* **Durable**: Once committed, the transaction is permanent and survives system crashes (stored on non-volatile memory).

> [!TIP] Exam PowerUP: RDS & Scaling
> If an exam question mentions **ACID**, it almost always refers to **RDS (Relational)**. Note that enforcing ACID properties limits the horizontal scalability of a database.

---

### 🚀 BASE (Focus: Availability & Scaling)
Modern NoSQL databases (like DynamoDB) often prioritize scale and speed over immediate consistency.

* **B**asically **A**vailable: Read and Write operations are available as much as possible, but without immediate consistency guarantees.
* **S**oft State: The database does not enforce consistency; this responsibility is offloaded to the application or user.
* **E**ventually Consistent: If we stop writing to the system, eventually all reads will return the same, most recent value.

> [!TIP] Exam PowerUP: NoSQL & DynamoDB
> * **BASE** usually refers to **NoSQL** databases (DynamoDB).
> * **Exception**: If you see "NoSQL" or "DynamoDB" mentioned alongside **ACID**, the question is likely referring to **DynamoDB Transactions**, which provide ACID compliance for complex operations.
