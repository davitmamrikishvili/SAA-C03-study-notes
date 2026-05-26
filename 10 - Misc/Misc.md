---
tags:
  - exam/misc
  - saa-c03
category: Exam Preparation
---

# 📦 Misc — SAA-C03 Gap Coverage

> [!INFO] Purpose
> This note covers AWS services and topics relevant to the **SAA-C03 exam** that are **not yet covered** in the main vault notes. It is organized by logical domain and kept concise — each entry includes the service's **core purpose** and the **exam-significant facts** only.

---

## 🏛️ Organizations & Account Governance

### AWS Control Tower
* **What**: Orchestration layer on top of AWS Organizations that automates the setup of a well-architected, multi-account AWS environment (a **Landing Zone**).
* **Key Concepts**:
    * **Guardrails**: Rules applied across accounts. Two types:
        * **Preventive**: Enforced via SCPs (e.g., "Disallow S3 bucket public read access").
        * **Detective**: Monitored via AWS Config rules (e.g., "Detect if CloudTrail is disabled").
    * **Account Factory**: Automated template for provisioning new, pre-configured member accounts.
* **Exam Triggers**: "Landing Zone," "govern multi-account environment," "preventive/detective guardrails" → **Control Tower**.

### AWS Service Catalog
* **What**: Allows organizations to create and manage curated catalogs of approved IT services (CFN templates, AMIs, software) that users can deploy.
* **Key Concepts**:
    * **Portfolio**: Collection of **products** + configuration.
    * **Constraints**: Limit what parameters a user can change (e.g., only allow `t3.micro` in dev).
* **Exam Triggers**: "Curated list of approved services," "self-service deployment with guardrails" → **Service Catalog**.

### AWS Resource Access Manager (RAM)
* **What**: Share AWS resources (Subnets, TGWs, License Configurations, Route 53 Resolver rules) with other accounts in your Organization — **without creating duplicate resources**.
* **Key Fact**: Resources are shared, not copied. The owner account retains ownership and control.
* **Exam Triggers**: "Share a VPC/subnet with another account," "share Transit Gateway" → **RAM**.

### AWS Cost Management
* **AWS Budgets**: Set custom budgets for cost, usage, or RI/Savings Plan coverage. Sends alerts via SNS/email when thresholds are breached (actual or forecasted).
* **Cost Explorer**: Visualize, understand, and forecast AWS costs & usage. Built-in **default reports**; **RI Utilization & Coverage reports**.
* **Cost Allocation Tags**: `aws:` (AWS-defined, auto-applied) and `user:` (you define). Must be **activated** before they appear in Cost Explorer. Tags take up to **24 hours** to propagate.
* **AWS Compute Optimizer**: ML-powered service that recommends **right-sized** EC2 instance types, EBS volumes, and Lambda functions based on utilization data.
* **Exam Triggers**: "Alert before spend exceeds X" → **Budgets**. "Analyze cost trends over time" → **Cost Explorer**. "Right-size EC2" → **Compute Optimizer**.

### AWS Trusted Advisor
* **What**: Automated best-practice inspection tool across five pillars: **Cost Optimization**, **Performance**, **Security**, **Fault Tolerance**, **Service Limits**.
* **Tiers**: Basic (7 core checks, free); Business/Enterprise (full set, programmatic access via API, CloudWatch integration).
* **Exam Triggers**: "Security best practices check," "service limit warnings," "unused resources" → **Trusted Advisor**.

### AWS Health Dashboard
* **What**: Provides information about AWS service health and events affecting your account.
* **Two Views**:
    * **Service Health Dashboard (SHD)**: Public view of all AWS services globally.
    * **Personal Health Dashboard (PHD)**: Personalized view of events affecting *your* specific resources. Integrates with EventBridge for automated response.
* **Exam Triggers**: "Alert on maintenance affecting your resources," "personalized health events" → **Personal Health Dashboard**.

---

## 🖥️ Additional Compute Services

### AWS Elastic Beanstalk
* **What**: Platform as a Service (PaaS) — deploys and scales web applications (Java, .NET, PHP, Node.js, Python, Ruby, Go, Docker). You provide code; Beanstalk automatically handles capacity, load balancing, scaling, and health monitoring.
* **Key Fact**: You **retain full control** of the underlying resources (EC2, ASG, ELB). Not a black box — you can SSH into instances and customize them.
* **Exam Triggers**: "PaaS," "upload code and let AWS manage infrastructure," "no infrastructure decisions" → **Elastic Beanstalk**.

### AWS Lightsail
* **What**: Simplified VPS (Virtual Private Server) offering. Pre-configured bundles (compute + storage + networking) for small-scale, low-complexity workloads. Fixed monthly pricing.
* **Exam Triggers**: "Simple VPS," "predictable pricing," "small website/WordPress" → **Lightsail**.

### AWS Batch
* **What**: Fully managed batch processing at any scale. Dynamically provisions EC2 or Fargate resources based on job volume.
* **Key Concepts**:
    * **Job**: Unit of work (shell script, executable, Docker image).
    * **Job Queue**: Jobs are submitted here.
    * **Compute Environment**: Managed (AWS handles infra) or Unmanaged (you manage). Spot instances are natively supported.
* **Exam Triggers**: "Batch processing," "run thousands of batch jobs," "dynamically provision compute" → **AWS Batch**.

### AWS Outposts
* **What**: AWS-managed hardware rack deployed **on-premises**. Runs a limited set of AWS services locally (EC2, EBS, S3, ECS, EKS, RDS). Acts as an extension of an AWS Region.
* **Key Fact**: Great for workloads needing **low latency to on-prem systems** or **local data processing requirements**.
* **Exam Triggers**: "AWS on-premises," "local data residency," "hybrid with AWS APIs" → **Outposts**.

### AWS Wavelength
* **What**: Embeds AWS compute and storage services within **5G networks** at telecom providers' edge locations. Ultra-low single-digit millisecond latency for mobile/connected devices.
* **Exam Triggers**: "5G," "ultra-low latency at the edge for mobile users," "telco edge" → **Wavelength**.

### AWS Local Zones
* **What**: An extension of an AWS Region that places compute, storage, and database services closer to large population centers. Provides **single-digit millisecond latency** to local users.
* **Key Difference from Wavelength**: Local Zones serve **any application** needing low latency; Wavelength is specifically for **5G mobile edge** use cases.
* **Exam Triggers**: "Low-latency to a specific city not served by a region," "extend your VPC to a local zone" → **Local Zones**.

### EC2 Hibernate
* **What**: Saves the contents of RAM to EBS root volume before stopping. On restart, RAM reloads — the OS and applications resume in the state they were in.
* **Constraints**: RAM < 150 GB. Root volume must be encrypted EBS. Max hibernation: 60 days. Not available for Spot instances.
* **Exam Triggers**: "Preserve in-memory state during stop," "fast boot-up" → **Hibernate**.

### EC2 Nitro System
* **What**: Underlying hardware virtualization platform for modern EC2 instances. Offloads networking, storage, and management to dedicated hardware — freeing up CPU for workloads.
* **Key Benefits**: Higher performance, better security, supports bare-metal instances.
* **Exam Triggers**: Rarely tested directly; mostly context about modern instance capabilities.

### Network Interfaces: ENI vs. ENA vs. EFA
| Feature             | ENI (Standard)              | ENA (Enhanced Networking)        | EFA (Elastic Fabric Adapter)        |
| :------------------ | :-------------------------- | :------------------------------- | :---------------------------------- |
| **Purpose**         | Basic networking            | High-bandwidth, low-latency      | HPC machine learning, MPI workloads |
| **Throughput**      | Up to 100 Gbps (varies)     | Up to 100 Gbps                   | Up to 100 Gbps                      |
| **OS Bypass (HPC)** | ❌ No                        | ❌ No                             | ✅ Yes (OS-bypass, low-latency)      |
| **Use Case**        | General networking          | High PPS workloads               | Tightly-coupled HPC clusters        |

### EC2 Image Builder
* **What**: Automates the creation, testing, and distribution of AMIs and container images. Pipeline-based: Build → Test → Distribute.
* **Exam Triggers**: "Automate AMI creation pipeline," "golden image pipeline" → **Image Builder**.

### Amazon ECR (Elastic Container Registry)
* **What**: Fully managed Docker container registry. Integrates natively with ECS and EKS. Supports image scanning for vulnerabilities and immutable image tags.
* **Exam Triggers**: Usually tested as a dependency of ECS/EKS rather than standalone.

### Amazon EKS (Elastic Kubernetes Service)
* **What**: Fully managed Kubernetes. Runs on EC2 or Fargate. AWS manages the control plane.
* **Key Fact**: **ECS is usually the preferred answer** on SAA-C03 unless the question specifically mentions "Kubernetes" or "K8s" as a requirement.
* **Exam Triggers**: "Kubernetes" → **EKS**.

---

## 🗄️ Additional Database & Analytics Services

### Amazon DocumentDB
* **What**: Fully managed, MongoDB-compatible document database (NoSQL). Automatically scales storage; replicates 6 copies across 3 AZs (like Aurora).
* **Exam Triggers**: "MongoDB-compatible," "managed MongoDB workloads" → **DocumentDB**.

### Amazon Neptune
* **What**: Fully managed **graph database** optimized for highly connected data and complex relationship queries.
* **Use Cases**: Social networks, recommendation engines, fraud detection, knowledge graphs.
* **Exam Triggers**: "Graph database," "relationships between entities," "connected data" → **Neptune**.

### Amazon QLDB (Quantum Ledger Database)
* **What**: Immutable, cryptographically verifiable **ledger database**. Every change is recorded in an append-only journal that cannot be modified or deleted.
* **Key Fact**: QLDB is **not** a blockchain — it's a centralized ledger owned by a single authority. Use **Amazon Managed Blockchain** for decentralized solutions.
* **Exam Triggers**: "Immutable ledger," "auditable history of changes," "cryptographic verification" → **QLDB**.

### Amazon Timestream
* **What**: Serverless, fully managed **time-series database** optimized for IoT and operational applications. Automatically tiers data (recent → memory, historical → magnetic).
* **Exam Triggers**: "Time-series," "IoT data," "sensor telemetry" → **Timestream**.

### AWS Glue
* **What**: Serverless **ETL (Extract, Transform, Load)** service. Discovers data, transforms it, and loads it into a target.
* **Key Concepts**:
    * **Glue Data Catalog**: Central metadata repository used by Athena, Redshift Spectrum, EMR (schema definitions for data in S3).
    * **Glue Crawlers**: Automatically scan data sources and populate the Data Catalog.
    * **Glue Jobs**: Execute the actual ETL logic (Spark or Python).
* **Exam Triggers**: "Serverless ETL," "data catalog," "schema discovery" → **Glue / Glue Data Catalog**.

### AWS Lake Formation
* **What**: Service to set up a **data lake** on S3 in days. Centralizes permissions (column/row-level) across Glue, Athena, Redshift, EMR, QuickSight.
* **Exam Triggers**: "Data lake," "centralized data permissions" → **Lake Formation**, not just S3 + Glue.

### Amazon EMR (Elastic MapReduce)
* **What**: Managed big data framework (Apache Spark, Hadoop, HBase, Presto, Flink, Hive) running on EC2 or EKS.
* **Exam Triggers**: "Hadoop," "Spark," "big data cluster" → **EMR**.

### Amazon MSK (Managed Streaming for Apache Kafka)
* **What**: Fully managed **Apache Kafka** — real-time streaming ingestion and processing. Alternative to Kinesis when existing Kafka tooling/skills requires migration.
* **Key Difference**: SAA-C03 default answer for **streaming** is usually **Kinesis** unless the question explicitly mentions "Kafka" or "migrating an existing Kafka cluster."
* **Exam Triggers**: "Apache Kafka," "Kafka migration" → **MSK**.

### Amazon QuickSight
* **What**: Serverless, fully managed **Business Intelligence (BI)** service. Create interactive dashboards and visualizations.
* **Key Feature**: **SPICE** — in-memory calculation engine that provides fast, parallel query performance.
* **Exam Triggers**: "BI dashboard," "data visualization," "business intelligence" → **QuickSight**.

### Amazon RDS Proxy
* **What**: Fully managed, highly available database proxy for RDS and Aurora. Sits between your application and the database, managing **connection pooling**.
* **Key Benefits**:
    * Reduces connection overhead — critical for Lambda and other serverless apps where each invocation opens a connection.
    * **Seamless failover** — during Multi-AZ failover, proxy re-routes connections, reducing failover time by up to 66%.
    * Enforces IAM database authentication.
* **Exam Triggers**: "Lambda connecting to RDS," "connection pooling," "reduce failover time" → **RDS Proxy**.

> [!TIP] Connecting the Lambda + RDS Pattern
> Without RDS Proxy, a Lambda function opens a new DB connection per invocation. At scale, this exhausts DB connections. Use **RDS Proxy** to pool and reuse connections, making serverless-to-RDS architectures viable.

---

## 📨 Additional Application Integration

### Amazon MQ
* **What**: Managed message broker service for **Apache ActiveMQ** and **RabbitMQ**. Used when you need industry-standard protocols (AMQP, MQTT, STOMP, JMS) rather than the proprietary SQS/SNS protocol.
* **Key Decision Rule**: **SQS/SNS** → cloud-native, decoupled, standard. **Amazon MQ** → migrating existing message broker to AWS or need standard protocols.
* **Exam Triggers**: "ActiveMQ," "RabbitMQ," "JMS," "AMQP," "lift-and-shift message broker" → **Amazon MQ**.

### AWS AppFlow
* **What**: Fully managed integration service for transferring data between **SaaS applications** (Salesforce, Zendesk, ServiceNow, Slack) and AWS services (S3, Redshift) bi-directionally.
* **Exam Triggers**: "SaaS integration," "Salesforce to S3," "sync data from SaaS app" → **AppFlow**.

### EventBridge — Architecture Deep Dive
Already introduced in CloudWatch. Key exam details beyond basic event routing:
* **Schema Registry**: Automatically discovers and stores event schemas, enabling code generation for type-safe event handling.
* **Event Replay**: Replay past events (up to archive storage) for testing or error recovery.
* **Cross-Account Events**: Send events from one AWS account to another — crucial for centralized event processing architectures.
* **API Destinations**: Route events directly to external HTTP APIs (e.g., third-party SaaS webhooks).
* **Exam Triggers**: "Replay events," "cross-account event bus," "SaaS integration" → **EventBridge**.

---

## 🔧 Developer Tools (CI/CD)

> [!INFO] The Big Four
> These four services form AWS's native CI/CD pipeline. They are typically used together.

| Service            | What it Does                                        | Trigger / Role                                       |
| :----------------- | :-------------------------------------------------- | :--------------------------------------------------- |
| **CodeCommit**     | Managed Git repository (like GitHub).               | Stores source code.                                  |
| **CodeBuild**      | Compiles code, runs tests, produces artifacts.      | "Build" stage. Fully managed; no build server needed. |
| **CodeDeploy**     | Automates code deployment to EC2, Lambda, ECS, on-prem. | Supports **Blue/Green** and **Canary** deployments.   |
| **CodePipeline**   | Orchestrates the entire CI/CD workflow.             | Connects CodeCommit → CodeBuild → CodeDeploy.         |

### Additional Developer Services
* **CodeArtifact**: Managed artifact repository (Maven, npm, pip, NuGet). Internal dependency management for your organization.
* **CodeGuru**: ML-powered code review (**Reviewer**) and application performance profiling (**Profiler**). Finds bugs, security issues, and optimization opportunities.
* **Cloud9**: Cloud-based IDE accessible via browser. Integrates with CodeStar.
* **Exam Triggers**: "CI/CD pipeline," "automated build/test/deploy" → **CodePipeline + CodeBuild + CodeDeploy**. "Git repository" → **CodeCommit**.

---

## 📋 Management & Governance

### AWS Well-Architected Framework
* **Six Pillars**: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, **Sustainability** (newest).
* **Well-Architected Tool**: Free tool that reviews workloads against the framework and provides improvement recommendations.
* **Exam Triggers**: "Review architecture against best practices," "Well-Architected review" → **Well-Architected Tool**.

### AWS Backup
* **What**: Centrally manage and automate backups across AWS services (EC2/EBS, RDS, Aurora, DynamoDB, EFS, FSx, S3, Storage Gateway).
* **Key Concepts**:
    * **Backup Plans**: Define frequency, retention, lifecycle.
    * **Vaults**: Store and encrypt backups. Can use **Vault Lock** (WORM — Write Once Read Many) for compliance.
    * **Cross-Region & Cross-Account**: Back up to another region or account.
* **Exam Triggers**: "Centralized backup management," "automated cross-region backups" → **AWS Backup**.

### Resource Groups & Tag Editor
* **Resource Groups**: Organize AWS resources by tags or CloudFormation stack. Useful for bulk operations (e.g., monitoring, patching).
* **Tag Editor**: Search and bulk-edit tags across all resources in all regions.
* **Exam Triggers**: "Organize resources for bulk operations," "apply tags globally" → **Tag Editor**.

### AWS AppConfig
* **What**: Deploy application configuration changes (feature flags, operational toggles) across EC2, Lambda, ECS, EKS, on-prem.
* **Key Concepts**:
    * **Validators**: Syntax and runtime validators check config before deployment.
    * **Deployment Strategies**: Linear, Canary, or All-at-Once rollouts.
* **Exam Triggers**: "Feature flags," "dynamic configuration," "controlled config rollout" → **AppConfig**.

---

## 🌐 Additional Networking

### Route 53 Resolver (Inbound/Outbound Endpoints)
* **Inbound Endpoints**: Allow on-prem DNS resolvers to forward queries to the VPC's Route 53 Resolver (e.g., to resolve private hosted zone records from on-prem).
* **Outbound Endpoints**: Allow VPC resources to forward DNS queries to on-prem DNS servers (conditional forwarding).
* **Exam Triggers**: "Resolve on-prem DNS from VPC (or vice versa)" → **Route 53 Resolver Endpoints**.

### AWS Network Firewall
* **What**: Managed stateful firewall and intrusion detection/prevention **for VPCs**. Filters traffic at the subnet boundary level (stateful L3-L7 inspection).
* **Comparison**: NACLs are stateless, SGs are stateful at the instance level. **Network Firewall** provides centralized, VPC-wide stateful inspection — sits between subnets and the IGW/VGW/TGW.
* **Exam Triggers**: "Stateful VPC firewall," "deep packet inspection for VPC," "domain-based filtering" → **Network Firewall**.

### VPC Traffic Mirroring
* **What**: Copies network traffic from an ENI and sends it to security/monitoring appliances for deep inspection. No agents or packet sniffers needed on instances.
* **Exam Triggers**: "Copy network traffic for inspection," "packet capture without agents" → **Traffic Mirroring**.

### AWS Client VPN
* **What**: Managed VPN service that allows individual users to securely connect to AWS or on-premises networks using OpenVPN-based clients.
* **Key Difference**: **Site-to-Site VPN** connects whole networks. **Client VPN** connects individual users (laptops, remote workers).
* **Exam Triggers**: "Remote users connecting to VPC," "individual VPN access" → **Client VPN**.

### ACM & Private Certificate Authority (PCA)
* **ACM**: Public and private SSL/TLS certificates for ALB, CloudFront, API Gateway, etc. Free for public certs. Auto-renews.
* **ACM PCA**: Managed private CA. Issue private certificates for internal HTTPS, IoT, or mutual TLS within your organization.
* **Critical Exam Reminder**: CloudFront certificates **must** be in `us-east-1`. ACM cannot deploy certs to EC2 directly.
* **Exam Triggers**: "Internal/private certificates," "private CA" → **ACM PCA**.

### Elastic IP Addresses
* **Limit**: 5 Elastic IPs per account per region (soft limit).
* **Best Practice**: Avoid using EIPs. Use DNS names (ELB, CloudFront) or Global Accelerator static IPs.
* **Exam Triggers**: "Five EIP limit," "avoid public IPs" → best practice reminder.

---

## 🔒 Additional Security Services

### AWS Security Hub
* **What**: Centralized security dashboard aggregating findings from GuardDuty, Inspector, Macie, IAM Access Analyzer, Firewall Manager, and third-party tools. Provides a unified security score (the **Security Score**).
* **Exam Triggers**: "Centralized security view," "single dashboard for security findings" → **Security Hub**.

### Amazon Detective
* **What**: Investigative service that analyzes and visualizes security data (GuardDuty findings, CloudTrail, VPC Flow Logs) to identify the **root cause** of security issues.
* **Exam Triggers**: "Root cause analysis of security incident," "visualize security data" → **Detective**.

### IAM Identity Center (formerly AWS SSO)
* **What**: Single sign-on for multiple AWS accounts and business applications. Integrates with your existing identity source (Active Directory, external IdP like Okta, or built-in directory).
* **Key Concepts**: Permission Sets define what users can do across accounts. Users sign in once and can access multiple accounts without separate credentials.
* **Exam Triggers**: "SSO across multiple accounts," "centralized user access," → **IAM Identity Center**.

### IAM Access Analyzer
* **What**: Identifies resources in your account that are shared with external entities (other accounts, organizations, or the public). Helps enforce **least privilege**.
* **Exam Triggers**: "Identify resources shared externally," "least privilege analysis" → **IAM Access Analyzer**.

### AWS Artifact
* **What**: Self-service portal for on-demand access to AWS compliance reports (SOC, PCI, ISO) and agreements (BAA for HIPAA).
* **Exam Triggers**: "Compliance reports," "SOC/PCI documentation" → **AWS Artifact**.

### STS — Important API Calls
| API Call                     | Purpose                                                          |
| :--------------------------- | :--------------------------------------------------------------- |
| `sts:AssumeRole`             | Assume an IAM role (temporary creds). Most common.              |
| `sts:AssumeRoleWithSAML`     | SAML federation (e.g., Active Directory → AWS).                  |
| `sts:AssumeRoleWithWebIdentity` | Web identity federation (Google, Facebook, Cognito).          |
| `sts:GetSessionToken`        | Short-term MFA-authenticated tokens for IAM users.               |
| `sts:GetCallerIdentity`      | Returns the identity (user/role) making the API call.            |

---

## 🗃️ Additional S3 Features

### S3 Access Points
* **What**: Named network endpoints with dedicated access policies, attached to a single bucket. Each access point enforces its own permissions — ideal for complex, multi-team bucket access.
* **Exam Triggers**: "Granular bucket access per team/application," "individual access policies per use case" → **S3 Access Points**.

### S3 Object Lambda
* **What**: Modify/transform object data **as it is retrieved** from S3 — without maintaining a separate, transformed copy. Uses a Lambda function to process the `GET` request inline.
* **Example**: Redact PII from reports on-the-fly, convert XML → JSON, dynamically resize images.
* **Exam Triggers**: "Transform object on retrieval," "redact/reshape data at read time" → **S3 Object Lambda**.

### S3 Multi-Region Access Points
* **What**: A single global endpoint that routes S3 requests to the **nearest** bucket with the lowest latency. Under the hood, uses AWS Global Accelerator.
* **Use Case**: Multi-region applications needing automatic regional failover for S3.
* **Exam Triggers**: "Single endpoint for multi-region S3," "automatic S3 regional routing" → **Multi-Region Access Points**.

---

## 📊 Additional Monitoring & Logging

### AWS X-Ray
* **What**: Distributed tracing service. Visualizes requests as they travel through your application (API Gateway → Lambda → DynamoDB → S3), identifying bottlenecks and errors.
* **Key Concepts**: **Segments** (service-level), **Subsegments** (downstream calls), **Traces** (end-to-end), **Annotations** (user-defined metadata for search/filtering).
* **Exam Triggers**: "Trace requests across microservices," "identify performance bottlenecks," "end-to-end request visualization" → **X-Ray**.

### CloudWatch Synthetics
* **What**: Configurable **canaries** (Node.js/Python scripts) that run on a schedule to monitor your endpoints and APIs. Simulates user behavior to detect issues before users do.
* **Blueprints**: Heartbeat monitor, API canary, GUI workflow builder, broken link checker.
* **Exam Triggers**: "Monitor endpoint availability externally," "canary testing," "proactive API health check" → **Synthetics**.

### CloudWatch Contributor Insights
* **What**: Analyzes time-series data to identify **top contributors** (e.g., the top-N talkers, the worst-performing URLs).
* **Exam Triggers**: "Identify top talkers," "find worst-performing URLs," "top-N analysis" → **Contributor Insights**.

### CloudWatch Application Insights
* **What**: Automated setup of monitoring, alarms, and dashboards for enterprise applications (SAP, .NET, SQL Server). Discovers resources, detects problems, and provides root-cause analysis.
* **Exam Triggers**: "Automated monitoring for .NET/SAP apps," "application-level observability" → **Application Insights**.

### CloudTrail Insights
* **What**: ML-powered anomaly detection on CloudTrail management events. Automatically flags unusual API activity (e.g., spikes in resource creation, unusual IAM actions).
* **Exam Triggers**: "Detect unusual API patterns," "anomaly in CloudTrail" → **CloudTrail Insights**.

---

> [!TIP] Exam PowerUP: Quick Decision Table
>
> | Scenario Keyword                        | Likely Answer                |
> | :-------------------------------------- | :--------------------------- |
> | Landing Zone / Guardrails               | **Control Tower**            |
> | Curated service catalog                 | **Service Catalog**          |
> | Share resources across accounts         | **RAM**                      |
> | Budget alerts                           | **AWS Budgets**              |
> | Cost trends / analysis                  | **Cost Explorer**            |
> | Best-practice checks                    | **Trusted Advisor**          |
> | Platform as a Service (PaaS)            | **Elastic Beanstalk**        |
> | Simple VPS / fixed-cost                 | **Lightsail**                |
> | Batch jobs at scale                     | **AWS Batch**                |
> | On-premises AWS rack                    | **Outposts**                 |
> | 5G edge computing                       | **Wavelength**               |
> | Low latency to city                     | **Local Zones**              |
> | Preserve RAM during stop               | **Hibernate**                |
> | HPC / MPI workloads                     | **EFA**                      |
> | Kubernetes                              | **EKS**                      |
> | MongoDB-compatible                      | **DocumentDB**               |
> | Graph database                          | **Neptune**                  |
> | Immutable ledger                        | **QLDB**                     |
> | Time-series IoT data                    | **Timestream**               |
> | Serverless ETL / Data Catalog           | **Glue**                     |
> | Data lake governance                    | **Lake Formation**           |
> | Hadoop / Spark / Big Data               | **EMR**                      |
> | Apache Kafka migration                  | **MSK**                      |
> | BI dashboards                           | **QuickSight**               |
> | DB connection pooling for Lambda        | **RDS Proxy**                |
> | ActiveMQ / RabbitMQ migration           | **Amazon MQ**                |
> | SaaS integration (Salesforce)           | **AppFlow**                  |
> | CI/CD pipeline orchestration            | **CodePipeline**             |
> | Centralized backup management           | **AWS Backup**               |
> | Feature flags / config deployment       | **AppConfig**                |
> | DNS between VPC and on-prem             | **Route 53 Resolver**        |
> | Stateful VPC firewall                   | **Network Firewall**         |
> | Copy ENI traffic for inspection         | **Traffic Mirroring**        |
> | Individual remote user VPN              | **Client VPN**               |
> | Security findings dashboard             | **Security Hub**             |
> | Root cause of security event            | **Detective**                |
> | SSO across multiple accounts            | **IAM Identity Center**      |
> | Resources shared externally             | **IAM Access Analyzer**      |
> | Compliance reports (SOC/PCI)            | **AWS Artifact**             |
> | Transform S3 objects on GET             | **S3 Object Lambda**         |
> | Distributed tracing                     | **X-Ray**                    |
> | Synthetic endpoint monitoring           | **CloudWatch Synthetics**    |
> | Unusual CloudTrail activity             | **CloudTrail Insights**      |

---

> [!INFO] Related Notes
> * For a complete directory, see the [[../README|README]].
> * For detailed ML service mappings, see: [[../09 - Machine Learning/Machine Learning|Machine Learning Overview]].
