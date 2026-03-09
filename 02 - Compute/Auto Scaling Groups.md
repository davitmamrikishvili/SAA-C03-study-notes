---
tags:
  - aws/ec2
  - aws/asg
  - compute
  - scaling
category: Compute
---

# 📈 Auto Scaling Groups & Launch Templates

> [!INFO] Overview
> **Auto Scaling Groups (ASGs)** work together with **Launch Templates (LTs)** and **Elastic Load Balancers (ELBs)** to deliver fully elastic, self-healing architectures on AWS. The ASG decides **WHEN** and **WHERE** to launch instances; the LT defines **WHAT** gets launched.

---

## 📋 Launch Configurations (LC) vs. Launch Templates (LT)

Both allow you to define the configuration of an EC2 instance in advance. They act as "blueprints" specifying:
* **AMI**, Instance Type & Size
* **Storage** & Key Pair
* **Networking** & Security Groups
* **User Data** & IAM Role

### Key Differences

| Feature               | Launch Configuration (LC)       | Launch Template (LT)                                                          |
| :-------------------- | :------------------------------ | :---------------------------------------------------------------------------- |
| **Editability**       | Not editable. Defined once.     | Not editable per-version, but supports **Versioning**.                        |
| **Usage**             | Can *only* be used within ASGs. | Can be used within ASGs **and** to launch instances directly (Console/CLI).   |
| **Advanced Features** | ❌ None.                         | ✅ T2/T3 Unlimited, Placement Groups, Capacity Reservations, Elastic Graphics. |
| **Status**            | Legacy.                         | **Recommended** by AWS.                                                       |

![[LCandLT.png]]

> [!WARNING] Exam Nugget
> Launch Templates provide **everything** Launch Configurations do and more. AWS recommends LTs for all new architectures. If an exam question gives you both as options, **always prefer Launch Template**.

---

## 🏗️ Auto Scaling Group (ASG) Architecture

ASGs provide **automatic scaling** and **self-healing** for EC2 instances.

### 🔢 The Three Core Values
Every ASG is defined by three capacity settings:

* **Minimum Size**: The floor. The ASG will never have fewer instances than this.
* **Desired Capacity**: The target. The ASG actively works to maintain this exact number of running instances.
* **Maximum Size**: The ceiling. The ASG will never launch more than this many instances.
* Example: `1:2:4` means Min=1, Desired=2, Max=4.

### 📍 Placement & Resilience
* ASGs run inside a **VPC** across one or more **subnets**.
* Whatever subnets are configured on the ASG are used to provision instances into.
* The ASG **tries to distribute instances evenly** across all configured subnets/AZs.

![[ASG-1.png]]

---

## 📊 Scaling Policies

Scaling Policies are rules that automatically adjust the ASG's Desired Capacity based on conditions. Scaling Policies are optional, thus ASGs can have none.

### 1. Manual Scaling
* You manually set the Desired Capacity.
* No automation—useful for testing, urgent situations or known, fixed-capacity events.

### 2. Scheduled Scaling
* Time-based adjustments defined in advance.
* **Use Case**: Scaling up before a known traffic event (e.g., a flash sale, business hours start) and scaling down afterwards.

### 3. Dynamic Scaling
Rules that react to real-time metrics (usually CloudWatch alarms) and adjust the ASG values.

* **Simple Scaling**: A basic threshold rule. After a scaling action, the ASG enters a **Cooldown Period** before it can act again.

![[ASG-Policies-1.png]]

* **Stepped Scaling**: A more granular version of Simple. The size of the adjustment changes based on the *magnitude* of the alarm breach. Does not have the same cooldown limitations. Recommended over the Simple Scaling.

![[ASG-Policies-2.png]]

* **Target Tracking**: The simplest to configure. You define a target metric value (e.g., *"Maintain average CPU at 40%"*), and the ASG automatically handles provisioning and terminating instances to stay at that target.

### 4. Scaling Based on SQS Queue
* A common decoupled pattern: a **Web Tier** pushes work items onto an **SQS Queue**, and a **Worker-Tier ASG** processes them.
* The ASG uses a custom CloudWatch metric — <span style="color:rgb(240, 75, 200)">ApproximateNumberOfMessagesVisible</span> — which reflects the number of messages waiting in the queue.
* As the queue depth grows, the ASG scales **out** to add more workers. As it shrinks, the ASG scales **in**.
* This is a powerful pattern because it **decouples** the producers from the consumers, allowing each side to scale independently.

### ⏳ Cooldown Periods
* A configurable waiting period after a scaling action before another action can take place.
* Prevents rapid, oscillating scaling events (known as "flapping").
* Default is **300 seconds** (5 minutes).

---

## 🩺 Health Checks & Self-Healing

* **Default**: ASGs use **EC2 Status Checks** to monitor instance health (checking hypervisor and system reachability).
* **ELB Health Checks**: When integrated with a Load Balancer, the ASG can use **Application-Aware** health checks from the ALB (e.g., checking for a `200 OK` response from the application endpoint).
* **Self-Healing**: When the ASG detects a health check failure, it **terminates** the unhealthy instance and provisions a brand-new replacement in its place.

> [!TIP] Simple Instance Recovery Pattern
> Create a Launch Template that automatically bootstraps an instance → set the ASG to use multiple subnets across different AZs → set capacity to `1:1:1`. If the instance fails, the ASG automatically replaces it, potentially in a different AZ.

---

## ⚖️ ASG + Load Balancer Integration

Instead of statically registering instances to an ELB's Target Group, you can integrate the ASG with that Target Group:

* **Auto-Registration**: As instances are launched by the ASG, they are **automatically added** to the Target Group.
* **Auto-Deregistration**: As instances are terminated, they are **automatically removed** from the Target Group.
* This combination of ASG + ELB is the foundation of a fully **elastic** architecture where the infrastructure dynamically adjusts to demand.

![[ASG-3.png]]

---

## ⚙️ Scaling Processes

ASGs have internal processes that can be individually **Suspended** or **Resumed** for fine-grained control:

| Process               | What it Controls                                                                                       |
| :-------------------- | :----------------------------------------------------------------------------------------------------- |
| **Launch**            | Ability to scale *out* (add instances). If suspended, no new instances are created.                    |
| **Terminate**         | Ability to scale *in* (remove instances). If suspended, no instances are terminated.                   |
| **AddToLoadBalancer** | Whether newly launched instances are automatically added to the associated Load Balancer/Target Group. |
| **AlarmNotification** | Whether the ASG reacts to CloudWatch alarms.                                                           |
| **AZRebalance**       | Whether the ASG redistributes instances evenly across all configured AZs.                              |
| **HealthCheck**       | Whether instance health checks are active across the ASG.                                              |
| **ReplaceUnhealthy**  | Whether the ASG replaces instances that are marked as unhealthy.                                       |
| **ScheduledActions**  | Whether the ASG will execute any scheduled scaling actions.                                            |
| **Standby**           | Allows you to put instances into an `InService` or `Standby` state (useful for maintenance).           |

---

> [!IMPORTANT] Exam PowerUP: ASG Nuggets
> * **Cost**: ASGs themselves are **free**. You only pay for the resources (EC2 instances) they create.
> * **Cooldowns**: Always use Cooldown Periods to prevent rapid, unnecessary scaling.
> * **Granularity**: Prefer **more, smaller instances** over fewer, larger ones for finer scaling control.
> * **Elasticity**: Use ASGs with **ALBs** for full elasticity and user abstraction.
> * **The Mantra**: ASG defines **WHEN** and **WHERE**. LT/LC defines **WHAT**.