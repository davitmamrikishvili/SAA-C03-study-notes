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

---

## 🌐 Networking Modes

Lambda has **two networking modes** that determine what resources a function can reach.

### 🌍 Public Networking (Default)
* By default, Lambda functions run with **public networking**. They can access public AWS services and the public internet.
* **Best performance** — no customer-specific VPC networking is required.
* **Trade-off**: Functions have **no access** to resources inside a VPC, unless those resources have public IPs and security rules that allow external access.

![[Lambda-2.png]]

### 🔒 VPC Networking
* A Lambda function can be configured to run inside a **private subnet** within your VPC.
* Once inside a VPC, the function **obeys all VPC networking rules** (NACLs, Security Groups).
* It can freely access other VPC-based resources (e.g., RDS, ElastiCache) as long as NACLs and Security Groups permit.
* **No internet access by default** — requires a **NAT Gateway + Internet Gateway** for public internet, or **VPC Endpoints** for access to public AWS services (S3, DynamoDB, etc.).
* **Permissions**: The Execution Role needs **EC2 network permissions** (`ec2:CreateNetworkInterface`, `ec2:DescribeNetworkInterfaces`, etc.) to attach an ENI in the VPC.

![[Lambda-4.png]]

> [!TIP] Exam Pointer: Lambda Networking
> * **"Lambda can't reach RDS"** → The function is likely in **Public mode**. Switch to **VPC Networking**.
> * **"Lambda in VPC can't reach the internet"** → Ensure a **NAT Gateway** exists in a public subnet with a route to an IGW.

---

## 🛡️ Security: Roles & Resource Policies

### Execution Role (Outbound Permissions)
* An IAM Role is created with a **Trust Policy** that trusts the Lambda service (`lambda.amazonaws.com`).
* When invoked, Lambda **assumes this role** and generates **temporary credentials**. These credentials are what the function's code uses to interact with other AWS resources (S3, DynamoDB, SQS, etc.).
* The scope of access is defined by the role's **Permissions Policy** — always apply **Least Privilege**.

### Resource Policy (Inbound Permissions)
* Similar in concept to an **S3 Bucket Policy**, a Lambda **Resource Policy** controls **who or what can invoke** the function.
* Use cases:
	* Allow an **external AWS account** to invoke the function.
	* Allow AWS services like **SNS**, **S3 Events**, or **API Gateway** to trigger the function.

![[Lambda-5.png]]

---

## 📋 Logging & Monitoring

* **CloudWatch Logs**: Stores the output/logs from Lambda executions (e.g., `print()` statements, errors).
* **CloudWatch Metrics**: Tracks invocation count, success/failure rates, retries, duration, and latency.
* **AWS X-Ray**: Provides **distributed tracing** to visualize the flow of requests across Lambda and other services.

> [!TIP] Exam Pointer: Logging Permissions
> Lambda **cannot** write logs, emit metrics, or send traces without the correct permissions. The **Execution Role** must include CloudWatch Logs and X-Ray write permissions (e.g., the `AWSLambdaBasicExecutionRole` managed policy).
