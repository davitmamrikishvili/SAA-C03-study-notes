---
tags:
  - aws/ec2
  - compute
  - automation
category: Compute
---

# 🚀 Advanced EC2

## 📥 EC2 User Data & Bootstrapping
> [!INFO] Definition
> **Bootstrapping** is the process of automatically configuring an EC2 instance or installing software when it first launches. This is achieved using **EC2 User Data**.

### 🛠️ Technical Mechanics
* **Execution**: User Data scripts are executed by the instance OS (e.g., `cloud-init` for Linux) **only during the initial launch**.
* **Opaque to AWS**: EC2 does not interpret the script. It simply passes the block of data to the OS. If the script fails, the instance will still show as "Running" (Silent Failure).
* **Accessibility**: User Data is injected into the instance and accessed via the Metadata IP: `http://169.254.169.254/latest/user-data`.
* **Persistence**: User Data can be modified while the instance is **stopped**, but it will not automatically re-run upon starting unless the OS is configured to do so.

### ⚠️ Constraints & Security
* **Size Limit**: Limited to **16 KB**.
* **Security**: User Data is **NOT secure**. It is stored in plain text. Never use it to pass long-term credentials (IAM roles are the solution for this).
* **Visibility**: Anyone with access to the instance OS or the EC2 Console can view the User Data.

---

## ⏱️ Boot-Time-To-Service-Time
This identifies how long it takes from the moment you request an instance until it is ready to serve traffic.

* **Provisioning Time**: The time AWS takes to allocate hardware and start the VM.
* **Post-Launch Time**: The time taken for software updates, installations, and configurations within the OS.

### 🍱 The Optimal Strategy: Baking vs. Bootstrapping
To minimize "Boot-Time-To-Service-Time," architects use a hybrid approach:

| Strategy          | When to Use                                                                             | Advantage                                                                 |
| :---------------- | :-------------------------------------------------------------------------------------- | :------------------------------------------------------------------------ |
| **AMI Baking**    | **Time-Intensive** tasks (Large app installs, OS hardening, base updates).              | Very fast launch times; everything is pre-installed.                      |
| **Bootstrapping** | **Dynamic** tasks (Applying final config, joining a cluster, fetching latest app code). | Extreme flexibility; the same AMI can be used for different environments. |

![[EC2Bootstrapping-2.png]]

> [!TIP] Exam PowerUP
> **Bake** part of the installation process that is time-intensive, **Bootstrap** the remaining dynamic tasks.

---

## 🛠️ CloudFormation Init (cfn-init)

While User Data is powerful, it is purely **procedural** and "fire-and-forget." For more complex, declarative configuration management, architects use **CloudFormation Init**.

### 🧩 Procedural (User Data) vs. Desired State (cfn-init)
* **User Data**: A script that runs line-by-line. If a command fails, the script continues (unless you handle errors manually). It is difficult to update or maintain.
* **cfn-init**: A configuration management system built into CloudFormation. You define the **Desired State** (what the instance *should* look like), and the `cfn-init` helper script makes it so.

### 🏗️ The 7 Sections of cfn-init
The `AWS::CloudFormation::Init` resource allows you to manage:
1. **Packages**: Install/Update software packages (RPM, MSI, Yum, etc.).
2. **Groups**: Create OS-level user groups.
3. **Users**: Create OS-level users.
4. **Sources**: Download and extract archives (tar, zip) into directories.
5. **Files**: Create files with specific content/permissions.
6. **Commands**: Execute shell commands in a specific order.
7. **Services**: Start/Stop/Disable OS services (ensure a service is always running).

### 🔄 How it Works
1. **Directives**: Configuration is stored in the **Metadata** section of the CloudFormation template under `AWS::CloudFormation::Init`.
2. **Trigger**: You call the `cfn-init` helper script from within the **User Data**.
3. **Execution**: `cfn-init` connects to CloudFormation, retrieves the metadata, and applies the configuration to the local instance.
4. **Updates**: Unlike User Data, `cfn-init` can be re-run during **Stack Updates** to modify the instance configuration without a total replacement.
![[CFN-INIT-1.png]]

---

## 🚦 CloudFormation Wait Conditions

To ensure an instance is strictly "Ready" before CloudFormation proceeds (e.g., before updating a Load Balancer), we use `cfn-signal` and a `CreationPolicy`.

### 📡 cfn-signal
* **Purpose**: A helper script used to send a "Success" or "Failure" signal back to CloudFormation.
* **Usage**: Typically placed at the very end of your User Data or `cfn-init` command list. It tells AWS: "The software is installed and the service is running."

### 🛡️ CreationPolicy
* **Purpose**: Tells the CloudFormation stack to **wait** for a specific number of signals before marking the resource as `CREATE_COMPLETE`.
* **Timeout**: You define a timeout period (e.g., 5 minutes). If the `cfn-signal` isn't received within that time, the stack fails and rolls back.

> [!TIP] Exam PowerUP: The Duo
> Always think of **cfn-signal** and **CreationPolicy** as a pair. One sends the signal (from the instance), and the other waits for it (in the CFN template). This prevents your stack from appearing "finished" while your application is still installing in the background.

![[CFN-INIT-2.png]]

---

## 🔑 EC2 Instance Roles

EC2 Instance Roles are the secure, best-practice way of providing AWS permissions to applications running on an EC2 instance without managing long-term credentials.

### 🏗️ Architecture: Role vs. Instance Profile
* **IAM Role**: The identity itself. It contains a **Permissions Policy** (what it can do) and a **Trust Policy** (which services, like `ec2.amazonaws.com`, can assume it).
* **Instance Profile**: A logical "container" or "wrapper" for the IAM Role. This is what you actually attach to the EC2 instance.
> [!NOTE] Console vs. CLI
>  When using the AWS Console, an Instance Profile is created and named automatically for you. When using the CLI or CloudFormation, you often have to manage them as separate resources.

### 📡 How it Works (Credential Delivery)
1. **Assumption**: The EC2 service assumes the role attached via the Instance Profile.
2. **IMDS Delivery**: Temporary security credentials (Access Key, Secret Key, and Session Token) are delivered into the instance via the **Instance Metadata Service (IMDS)**.
3. **Endpoint**: `http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>`
4. **Automatic Renewal**: AWS automatically rotates these credentials. Applications using the AWS SDK or CLI will automatically fetch and refresh these credentials from the metadata endpoint.

### 🛡️ Security Best Practices
* **Never Use Static Keys**: Never store `AWSAccessKeyId` or `AWSSecretAccessKey` in configuration files or Environment Variables on an EC2 instance.
* **Least Privilege**: Attach only the specific permissions required for the application.
* **Universal Support**: Most AWS-aware software and all official SDKs will automatically check for metadata credentials if no other credentials are found.

> [!TIP] Exam PowerUP: Troubleshooting Permissions
> If an application on EC2 gets an "Access Denied" error, check:
> 1. Is there an **Instance Profile** attached?
> 2. Does the **IAM Role** have the correct permissions?
> 3. Does the **Trust Policy** allow `ec2.amazonaws.com`?

---

## 📊 System and Application Logging on EC2

By default, CloudWatch only monitors **External Metrics** (CPU, Network, Disk I/O) that the hypervisor can see. It has **zero visibility** into the operating system or applications running inside the EC2 instance.

### 🕵️ The CloudWatch Unified Agent
To capture internal OS metrics (like Memory utilization) and application logs, you must install the **CloudWatch Unified Agent** on the instance.

* **OS Level Logs**: System logs (`/var/log/messages`, syslog), security logs, etc.
* **Application Logs**: Custom logs generated by your web server (Apache/Nginx), database, or app code.
* **Advanced Metrics**: High-resolution metrics such as **Memory usage**, Disk swap, and Disk space utilized.
![[LoggingonEC2CWAgent.png]]

### 🏗️ Hierarchy: Log Groups & Log Streams
CloudWatch Logs organizes data into a specific hierarchy:
* **Log Group**: Represents a "category" of logs (usually one per application or log file type, e.g., `/prd/webserver/access_log`). You define retention settings and permissions at this level.
* **Log Stream**: Represents a specific "source" within that group. In an Auto Scaling Group, each EC2 instance will have its own unique **Log Stream** within the shared Log Group.

### 🔐 Prerequisites & Permissions
1. **Installation**: The agent must be installed and configured (usually via a JSON config file).
2. **IAM Permissions**: The instance must have an **IAM Role (Instance Profile)** with permissions to upload data to CloudWatch (`logs:CreateLogStream`, `logs:PutLogEvents`, etc.).
> [!TIP] AWS Managed Policy
> Use the `CloudWatchAgentServerPolicy` for a quick and secure setup.

---

## 🏗️ EC2 Placement Groups

By default, AWS places EC2 instances across its infrastructure in a way that minimizes correlated failure. However, **Placement Groups** allow you to influence this logic to optimize for either **Ultra-High Performance** or **Maximum Infrastructure Resilience**.

### 📊 Comparison Table

| Feature          | Cluster (Performance)        | Spread (Resilience)          | Partition (Scalable Resilience) |
| :--------------- | :--------------------------- | :--------------------------- | :------------------------------ |
| **Primary Goal** | Low-latency, High Throughput | Physical hardware isolation  | Distributed fault domains       |
| **Scope**        | **Single AZ**                | Multi-AZ (Region)            | Multi-AZ (Region)               |
| **Limits**       | Restricted by AZ capacity    | **7 Instances per AZ**       | **7 Partitions per AZ**         |
| **Performance**  | **10 Gbps** single-stream    | Standard                     | Standard                        |
| **Use Case**     | HPC, Big Data, Ad-tech       | DBs, critical micro-clusters | Hadoop, Cassandra, HBase        |

### 🚀 Cluster Placement Groups (Performance)
Designed to minimize network latency and maximize data throughput by packing instances close together.

* **Physical Placement**: Instances are placed on the same physical rack and often sharing the same EC2 Host.
* **Networking**: Supports **10 Gbps** single-stream transfer rates (vs. standard 5 Gbps).
* **Constraint**: Locked to a **Single Availability Zone (AZ)**.
* **Best Practice**: Launch all instances at once and use the same instance type to ensure AWS can find a physical block of capacity large enough.
* **Trade-off**: Lowest resilience. A single rack/host failure impacts all instances.

![[PlacementGroups-1.png]]

### 🛡️ Spread Placement Groups (Resilience)
Ensures total physical isolation for a small number of critical instances.

* **Physical Placement**: Every instance is guaranteed to be on a **different rack**, with its own redundant power and network supply.
* **Availability**: Can span across multiple AZs.
* **Constraint**: Limited to **7 instances per AZ** (Hard Limit).
* **Restriction**: Does not support Dedicated Instances or Dedicated Hosts.
* **Use Case**: Individual nodes that must stay alive (e.g., Active/Passive DB nodes, Domain Controllers).

![[PlacementGroups-2.png]]

### 🧱 Partition Placement Groups (Scalable Resilience)
Designed for big data and distributed systems that handle their own data replication.

* **Physical Placement**: Instances are grouped into **Partitions**. Each partition uses its own set of racks. No sharing between partitions.
* **Availability**: Spans across multiple AZs.
* **Scale**: Supports a much higher instance count than Spread (up to 7 partitions per AZ, with many instances per partition).
* **Topology Awareness**: Ideal for apps like **HDFS (Hadoop), HBase, and Cassandra**. These apps know which partition data is in and ensure replicas are stored in *different* partitions to survive a rack failure.
* **Control**: You can manually place an instance in a specific partition or let AWS handle it automatically.

![[PlacementGroups-3.png]]

> [!TIP] Exam PowerUP: The Selection Logic
> * If the keyword is **"Latency"** or **"Single Stream Performance"** $\to$ Choose **Cluster**.
> * If the keyword is **"Hardware Isolation"** or **"Individual Racks"** $\to$ Choose **Spread**.
> * If the keyword is **"Topology Aware"** or **"Distributed Datastores (Cassandra)"** $\to$ Choose **Partition**.

---

## 🏢 EC2 Dedicated Hosts

An **EC2 Dedicated Host** is a physical server with EC2 instance capacity fully dedicated to your use. Unlike shared multi-tenant hosts, you have full visibility into the physical hardware (sockets and cores).

### 🔑 Key Characteristics
* **Pricing**: You pay for the **physical host** itself, not the individual instances. There are no per-instance hourly charges for instances running on the host. Available as On-Demand or via **Reserved Instances (HRI)**.
* **Hardware Control**: Useful for software that is licensed based on physical sockets, cores, or VM affinity (BYOL).
* **Isolation**: You can guarantee that all instances run on the same physical hardware, which is critical for some regulatory compliance.

![[DedicatedHosts-1.png]]

* On **Nitro-based hosts**, you can run different instance sizes from the same family on a single host.

![[DedicatedHosts-2.png]]

### 🛡️ Sharing and Collaboration
* **Resource Access Manager (RAM)**: You can share a Dedicated Host with other accounts within your AWS Organization.
* **Visibility**:
    * **Participant Accounts** can launch instances on the shared host but cannot see instances from other accounts.
    * **The Owner Account** can see all instances on the host but cannot stop/control instances managed by other accounts.

### 🚫 Important Restrictions
* **Supported AMIs**: Does **not** support certain paid AMIs (RHEL, SUSE, Windows via AWS License) because you must provide your own licenses.
* **Services**: Amazon **RDS** is not supported on Dedicated Hosts.
* **Networking**: **Placement Groups** are not supported for Dedicated Hosts.

---

## ⚡ Enhanced Networking (SR-IOV)

Enhanced Networking uses **Single Root I/O Virtualization (SR-IOV)** to provide high-performance networking with lower CPU utilization on the host.

![[EC2EnhancedNetworking.png]]
### 🧩 How it Works (SR-IOV)
1. **Standard Networking**: The EC2 Host acts as a middleware between the virtual NIC and the physical NIC. This creates "jitter" and consumes host CPU cycles.
2. **SR-IOV**: The physical Network Interface Card (NIC) is virtualization-aware. It presents multiple "logical" NICs (VFs - Virtual Functions) directly to the instances.
3. **Result**: The instance talks directly to the hardware. This allows for:
    * Higher bandwidth and **Higher Packets-Per-Second (PPS)**.
    * Significantly and consistently **lower latency**.
    * Lower host CPU overhead.

> [!INFO] Availability
> This feature is enabled by default (or available at no extra charge) on almost all modern EC2 instance types. It is a prerequisite for **Cluster Placement Groups**.

---

## 📦 EBS Optimized Instances

An **EBS-optimized instance** uses a dedicated network stack for EBS traffic, providing guaranteed performance between EC2 and EBS.
* **Dedicated Bandwidth**: It provides a dedicated "private lane" for storage traffic, ensuring that general network traffic (like web requests) doesn't compete for bandwidth with disk I/O.
* **Performance**: Guaranteed throughput (Mbps) and IOPS for the EBS volumes.
* **Modern vs. Legacy**:
    * On **modern instances**, this is enabled by default at **no extra cost**.
    * On older instance types, you may need to enable it manually, and it may carry an additional hourly cost.

> [!TIP] Exam PowerUP: Networking Trio
> * **Enhanced Networking**: Improves **Instance-to-Instance** (or Instance-to-Internet) speeds via SR-IOV.
> * **EBS Optimized**: Improves **Instance-to-Storage** performance by separating traffic.
> * **Elastic Fabric Adapter (EFA)**: Specialized for **HPC/MPI** workloads requiring node-to-node communication with lower latency than standard Enhanced Networking.