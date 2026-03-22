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

---

## 📡 Invocation Models

Lambda functions can be invoked in **three distinct ways**. Understanding the differences is critical for the exam.

| Model                    | Who waits? | Who retries?   | Example Triggers               |
| :----------------------- | :--------- | :------------- | :----------------------------- |
| **Synchronous**          | The caller | The caller     | CLI/SDK, API Gateway           |
| **Asynchronous**         | Nobody     | Lambda (×2)    | S3 Events, SNS, EventBridge    |
| **Event Source Mapping** | Nobody     | Lambda (batch) | Kinesis, DynamoDB Streams, SQS |

### ⚡ Synchronous Invocation
* The caller (CLI, SDK, or API Gateway) **invokes the function, passes data, and waits** for the response.
* The result (success or failure) is returned **directly** to the caller within the same request.
* **API Gateway pattern**: Client → API Gateway → Lambda → processing → response flows back through API Gateway → Client — all while the client waits.
* **Error handling**: The **caller is responsible** for any retry logic.

![[Lambda-9.png]]

### 🔄 Asynchronous Invocation
* Typically used when **AWS services** trigger Lambda (e.g., S3 puts an object, SNS publishes a message).
* The event is placed on an **internal queue**. Lambda manages polling, scaling, and retries.
* **Retry behavior**: Lambda automatically retries **twice** on failure (for a total of 3 attempts).
* **Idempotency requirement**: Because the same event can be processed multiple times, the function **must be idempotent** — reprocessing should produce the same end state.
* **Dead Letter Queue (DLQ)**: Events that fail all retries can be sent to an **SQS queue** or **SNS topic** for diagnostic processing.
* **Destinations**: Lambda supports sending successful or failed event results to destinations (SQS, SNS, EventBridge, or another Lambda).

![[Lambda-6.png]]

### 📬 Event Source Mapping
* Used for **streams and queues** that don't natively generate events to invoke Lambda — specifically **Kinesis Data Streams**, **DynamoDB Streams**, and **SQS**.
* The Event Source Mapping **reads/polls** from the source and delivers **event batches** to Lambda.
* **Batch processing**: An entire batch either **succeeds or fails** as a unit — there is no partial success.
* **Permissions**: The **Execution Role** is used by the Event Source Mapping to read from the source (not a resource policy).
* **Failed batches**: Can be sent to **SQS queues** or **SNS topics** for later analysis.

![[Lambda-7.png]]

---

## 🏷️ Versions & Aliases

* **Versions**: Each published version is a snapshot of the function's **code + configuration**. Once published, a version is **immutable** and receives its own unique **ARN**.
* <span style="color:rgb(240, 75, 200)">$Latest</span>: A pointer that always references the **most recent** (unpublished, mutable) version of the function.
* **Aliases**: Named pointers (e.g., `DEV`, `STAGE`, `PROD`) that map to a specific version. Aliases **can be changed** to point at a different version — enabling safe, controlled deployments (e.g., shift `PROD` from v3 → v4).

---

## 🚀 Startup Times (Cold & Warm Starts)

* **Execution Context**: Lambda code runs inside a runtime environment (also called an **execution context**).
* **Cold Start**: A **full creation** of the execution context — downloading the deployment package, initializing the runtime, and loading function code. This adds latency to the invocation.
* **Warm Start**: Lambda may **reuse** an existing execution context for subsequent invocations, skipping the setup overhead.
* **Context Expiry**: If too much time passes between invocations, Lambda **deletes the context**, causing the next invocation to cold start again.
* **Concurrency**: Each execution context handles **one invocation at a time**. If 20 concurrent invocations are needed, this could trigger **20 cold starts**.

### ⚙️ Provisioned Concurrency
* **Pre-warms** a specified number of execution contexts so they are ready before invocations arrive — **eliminating cold starts**.
* **Use cases**: Periods of predictable high load, or preparing for a new production release of a serverless application.

### `/tmp` Optimization
* You can pre-download data (e.g., ML models, reference files) into the `/tmp` space. If a subsequent invocation reuses the same execution context, the data is **already available** without re-downloading.
* **Critical rule**: Functions must **never assume** the presence of anything in `/tmp`. Always code defensively — treat every invocation as if it is running in a **completely fresh environment**.

![[Lambda-8.png]]

---

## 🔀 AWS Step Functions

> [!INFO] Why Step Functions?
> Lambda has hard limitations: **15-minute max** execution, **stateless** environments, and chaining functions manually **gets messy at scale**. Step Functions solve this by letting you build **state machines** — managed workflows with a clear start, end, and states in between.

### 🧩 Core Concepts
* **State Machine**: A workflow with a **Start point**, one or more **States**, and an **End point** (`START → STATES → END`).
* **States**: The building blocks of a workflow. Each state can take in data, process it, modify it, and output data to the next state.
* **Amazon States Language (ASL)**: State machines are defined as **JSON templates** using ASL.
* **Permissions**: An **IAM Role** is assumed by the state machine to interact with other AWS services.
* **Triggers**: Can be started via **API Gateway**, **IoT Rules**, **EventBridge**, or **Lambda**.

### ⚙️ Workflow Types

| Type         | Duration Limit | Best For                                     |
| :----------- | :------------- | :------------------------------------------- |
| **Standard** | **1 year**     | Long-running, auditable workflows (default). |
| **Express**  | **5 minutes**  | High-volume, event-processing workloads.     |

### 🎛️ State Types

| State              | Description                                                                                                |
| :----------------- | :--------------------------------------------------------------------------------------------------------- |
| **Task**           | Performs a **unit of work** — invokes a service (Lambda, Batch, DynamoDB, ECS, SNS, SQS, Glue, EMR, etc.). |
| **Choice**         | Takes a **different path** depending on input conditions (like an `if/else`).                              |
| **Parallel**       | Creates **parallel branches** that execute simultaneously within the state machine.                        |
| **Map**            | Iterates over a **list of items**, performing an action (or set of actions) on each.                       |
| **Wait**           | Pauses execution for a **specific duration** or until a **specific date/time**.                            |
| **Succeed / Fail** | **Terminal states** — the workflow ends with a success or failure result.                                  |