---
tags:
  - aws/ecs
  - containers
  - compute
category: Compute
---

# 🚢 ECS - Elastic Container Service

Amazon ECS is a fully managed container orchestration service that allows you to run, scale, and secure Docker containers on AWS.

---

## 🏗️ ECS Architecture & Components

### 🧱 Cluster
* A logical grouping of resources. Your containers run within a cluster.
* ECS orchestrates where and how containers run based on your configuration.

### 🖼️ ECR - Elastic Container Registry
* A fully managed Docker container registry used to store, manage, and deploy Docker container images.

### 📦 Container Definition
* Defined within a Task Definition.
* Specifies the **Docker image** to use, port mappings, and resource allocations (CPU/Memory).
* A single task can contain multiple container definitions (e.g., App + Sidecar).

### 📑 Task Definition (The Blueprint)
* A text file (JSON) that describes one or more containers that form your application.
* **Parameters**: CPU/Memory limits, Docker images, port mappings, and networking mode.
* **Task Role**: An IAM Role that the task assumes to gain temporary credentials for interacting with other AWS services.
> [!IMPORTANT] Exam PowerUP
> **Task Roles** are the best practice for giving containers permissions (not Instance Roles).

### 🚀 Task (The Running Instance)
* The instantiation of a Task Definition within a cluster.
* A Task represents the application as a whole and can contain one or more containers.
* Tasks themselves **do not** scale automatically or provide high availability.

### 🔄 ECS Service (The Controller)
* Used to run and maintain a specified number of instances of a task definition simultaneously.
* If a task fails, the Service scheduler launches another instance.
* Provides **High Availability** and **resilience**.
* Optional, but required for scaling and load balancing.

![[ECS.png]]

---

## 🛠️ Launch Types: EC2 vs. Fargate

> [!INFO] Shared Components
> The **Scheduling & Orchestration**, **Cluster Manager**, and **Placement Engine** exist in both modes and handle the placement and lifecycle of your containers.

| Characteristic | EC2 Mode | Fargate Mode (Serverless) |
| :--- | :--- | :--- |
| **Management** | You manage the EC2 instances (patching, scaling). | AWS manages the underlying infrastructure. |
| **Control** | Full control over the underlying EC2 host. | No access to the underlying host. |
| **Cost** | You pay for the EC2 instances, regardless of container usage. | You pay for the CPU and Memory resources consumed by the task. |
| **Scaling** | You must scale the EC2 cluster (ASG) AND the tasks. | You only scale the tasks. |

### 🖥️ EC2 Mode
* **VPC Integration**: The ECS Cluster is created within your VPC, benefiting from Multi-AZ resilience.
* **Container Instances**: EC2 instances are used as "Container Instances."
* **Management**: Handled via an **Auto-Scaling Group (ASG)**. While more flexible, it introduces more management overhead (patching, host security).

### 🛸 Fargate Mode
* **Serverless**: Removes almost all infrastructure management.
* **Shared Platform**: AWS maintains a shared Fargate platform, but provides **EC2-level isolation** between customers.
* **Networking**: Each task is injected into your VPC and given its own **ENI**.
* **Public Access**: If configured with public subnets/IPv4, tasks can be directly accessible from the internet.

### 🚀 Exam PowerUP: The Selection Hierarchy
1. **EC2 Native**: Virtual machine. Best for full OS control or legacy monoliths.
2. **ECS (EC2 Mode)**: High flexibility, cost-conscious for massive/steady-state workloads.
3. **ECS (Fargate Mode)**: Minimum overhead, best for bursty, small, or periodic (batch) workloads.