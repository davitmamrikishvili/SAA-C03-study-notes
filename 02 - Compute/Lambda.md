---
tags:
  - aws/lambda
  - compute
  - serverless
category: Compute
---

# ⚡ AWS Lambda

> [!INFO] Definition
> **AWS Lambda** is a **Function-as-a-Service (FaaS)** platform. You provide short-running, focused code, and Lambda handles all the infrastructure — provisioning, scaling, and patching. It is a key building block of **Serverless architectures**.

## 🏗️ Core Concepts

* **Lambda Function**: Think of it as your **code + all associated configuration** (runtime, memory, timeout, permissions). At its most basic, it is a **deployment package** that Lambda executes.
* **Runtime Environment**: When a function is invoked, Lambda downloads the deployment package and executes it within a temporary, isolated runtime environment.
* **Runtimes**: Functions are written in a specific language runtime (e.g., Python 3.12, Node.js 20, Java 21). You can also create **custom runtimes** (e.g., Rust) using **Lambda Layers**.
* **Stateless**: No data persists between invocations. Every invocation runs in a **brand new environment**.

---

## 📊 Resource Allocation & Limits

| Resource         | Details                                                              |
| :--------------- | :------------------------------------------------------------------- |
| **Memory**       | **128 MB** to **10,240 MB** (10 GB), in 1 MB increments.             |
| **vCPU**         | Allocated **indirectly** — scales proportionally with memory.        |
| **Temp Storage** | **512 MB** mounted at `/tmp` (configurable up to 10 GB).             |
| **Timeout**      | Maximum execution time per invocation: **900 seconds (15 minutes)**. |

* **Billing**: You pay for the **duration** a function runs × the **amount of memory** allocated. No charge when the function is idle.

> [!TIP] Exam Pointer: The 15-Minute Limit
> A single Lambda invocation **cannot** exceed 15 minutes. For longer-running workflows, use **AWS Step Functions** to orchestrate multiple Lambda invocations into a state machine.

---

## 🔐 Security: Execution Roles

* **Execution Role**: An **IAM Role** assumed by the Lambda function at runtime. This is how Lambda gets permissions to interact with other AWS services (S3, DynamoDB, SQS, etc.).
* **Best Practice**: Apply **Least Privilege** — only grant the specific permissions your function needs.

![[Lambda-1.png]]

> [!TIP] Exam Pointer: Docker & Lambda
> * If an exam question mentions **"Docker"** as a deployment model, it is generally an **anti-pattern for Lambda** — consider **ECS/Fargate** instead.
> * However, Lambda **does support container images** (up to 10 GB) that implement the Lambda Runtime API. Don't confuse "Docker containers on ECS" with "container images on Lambda."

---

## 🛠️ Common Use Cases

| Pattern                          | Services Involved                          |
| :------------------------------- | :----------------------------------------- |
| **Serverless Applications**      | S3 + API Gateway + Lambda                  |
| **File Processing**              | S3 Events → Lambda                         |
| **Database Triggers**            | DynamoDB Streams → Lambda                  |
| **Serverless CRON**              | EventBridge (CloudWatch Events) → Lambda   |
| **Real-time Stream Processing**  | Kinesis → Lambda                           |
