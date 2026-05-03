---
tags:
  - aws/cloudfront
  - networking
  - cdn
category: Networking
---

# 🌍 Amazon CloudFront

> [!INFO] Definition
> **Amazon CloudFront** is a global Content Delivery Network (CDN) service. Its primary job is to securely deliver data, videos, applications, and APIs to customers globally with low latency and high transfer speeds.

---

## 🏗️ CloudFront Architecture & Vocabulary

Understanding the core components of CloudFront is essential for designing resilient and performant delivery networks.

### 🎯 Core Components
* **Origin**: The true source location of your content (where the actual definitive files live).
    * **S3 Origin**: An Amazon S3 bucket.
    * **Custom Origin**: Any HTTP web server, such as an EC2 instance, Application Load Balancer (ALB), or an on-premises server.
* **Distribution**: The fundamental 'configuration' unit of CloudFront. This is what you create. It defines which origins to use and how content should be cached and delivered.
* **Edge Location**: A globally distributed site consisting of infrastructure designed to cache your data locally, close to the end-user.
* **Regional Edge Cache**: A larger, regional version of an Edge Location. It sits between your Origin and the Edge Locations to provide a broader, second layer of caching.
> [!WARNING] Exam Nugget
> **Only Custom Origins** use Regional Edge Caches. S3 Origins do not.

### ⚙️ How it Works
1. A user requests content. The request is routed via global DNS to the physically closest **Edge Location**.
2. If the Edge Location has a cached copy, it serves it immediately (Cache Hit).
3. If not (Cache Miss), it checks the **Regional Edge Cache** (if applicable).
4. If still missing, the request is forwarded to the **Origin** to fetch the original file.
5. Note: CloudFront provides **read-only caching**. Any write requests (like `POST` or `PUT`) are proxied directly back to the origin; they are not cached.

---

## 🚦 Behaviors & Routing

A Distribution isn't just a simple pass-through; you can configure complex routing logic using **Behaviors**.

![[CloudFrontArchitecture-3.png]]

* **What is a Behavior?**: A set of rules and configurations within a Distribution that dictates how specific types of requests should be handled.
* **Pattern Matching**: Behaviors operate on a **pattern match** principle (e.g., `*.jpg`, `/api/*`).
* **Evaluation**: When a request comes in, CloudFront evaluates the path against the configured Behaviors.
    * If a request matches a specific pattern, the settings for that Behavior are used (e.g., routing `/images/*` to an S3 origin).
    * If there is no match, the **Default Behavior** (which acts as a catch-all, e.g., `Default (*)`) is used.
* Every CloudFront Distribution **must** have at least one Default Behavior.

TTL and Invalidations
- More frequent cache hits = lower origin load
- Default TTL (this is defined on behaviour in distribution) is 24 hours
- You can set minimum TTL and maximum TTL
- some important headers you can define per object:
	- <span style="color:rgb(240, 75, 200)"><span style="color:rgb(240, 75, 200)">Cache-Control max-age</span></span> (seconds)
	- <span style="color:rgb(240, 75, 200)">Cache-Control s-maxage</span> (seconds)
	- <span style="color:rgb(240, 75, 200)">Expires</span> (Date & Time)
- Minimum and Maximum TTL defined on behavior are limiters on the headers. example: if object header value is less than the minimum TTL of the behaviour then the minimum TTL is used
- the architecture: you have default ttl on the behaviour, you can change this value (by default 24 hours), and this applies to any object that doesn't have a per object ttl set. You also set minimum TTL and maximum TTL values and these act as limiters for any per-object settings that are defined using these cache control headers.
- These headers can be set using Custom Origins or S3 (Via object metadata).
- Cache Invalidations are performed on a distribution. Whatever you set invalidation to be, it is applied to all edge locations within the distribution. So, it's not immediate.
- It immediately expries any objects regardless of their TTL based on the invalidation pattern you specify. example: /* -> this invalidates every cached object.
- There is a cost to invalidation, so it should not be used regurarly, rather to correct some mistakes.
- (IMPORTANT FOR EXAM) It's recommended to use versioned file names, examples: img_v1.jpg, img_v2.jgp, ... This way, invalidation won't be needed. Even the data cached in user's browser won't impact the image of the object the user will see. Also, logging is more effective, because you know which actual object was used. Also, it's less expensive, because you don't need to use continued cache invalidations.
- Don't confuse versioned file names with S3 object versioning: S3 object versioning allows you to have different data for an objecttl, different objects that use the same name. CloudFront  will always use the latest object version in the bucket by default. Using versioned file names means having different file names for different actual versions of an object. That means, each of these file names will be cached independently on every edge location and you can move between  them in a consistent way by making changes to your application.