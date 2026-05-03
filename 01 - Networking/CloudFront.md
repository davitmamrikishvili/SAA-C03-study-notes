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

---

## ⏱️ TTL and Invalidations

Maximizing cache hits reduces the load on your origin and improves user experience.

### ⚙️ TTL Settings on Behaviors
* **Default TTL**: Applied if the origin doesn't specify any cache control headers. The default is **24 hours**.
* **Minimum & Maximum TTL**: These act as **limiters** on any per-object headers coming from the origin.
    * *Example*: If an object's header says "cache for 1 hour", but the Behavior has a Minimum TTL of "4 hours", CloudFront forces the 4-hour minimum.

### 📝 Per-Object Caching Headers
You can achieve granular caching control by having your Custom Origin or S3 (via object metadata) send specific HTTP headers:
* <span style="color:rgb(240, 75, 200)">Cache-Control: max-age</span> (seconds)
* <span style="color:rgb(240, 75, 200)">Cache-Control: s-maxage</span> (seconds)
* <span style="color:rgb(240, 75, 200)">Expires</span> (Specific Date & Time)

### 🚨 Cache Invalidations
Sometimes you upload a new version of a file to your origin, but CloudFront continues to serve the old, cached version until the TTL expires.
* An **Invalidation** forces CloudFront to immediately expire objects across all Edge Locations regardless of their TTL.
* **Pattern Based**: You specify patterns (like `/*` to clear everything, or `/images/logo.png`).
* **Cost & Speed**: Invalidations are **not immediate** (it takes time to propagate to all edges) and they **cost money**. They should be used to correct mistakes, not as a standard deployment pipeline practice.

> [!IMPORTANT] Exam PowerUP: Versioned File Names
> AWS strongly recommends using **Versioned File Names** (e.g., `img_v1.jpg`, `img_v2.jpg`) instead of Cache Invalidations.
>
> * **No Invalidation Needed**: Because the filename changes, CloudFront treats it as a brand new object, avoiding invalidation delays and costs.
> * **Bypasses Browser Caching**: Local client browsers will request the new file instead of using their local cache.
> * **Better Logging**: Your access logs clearly show exactly which version of the asset was requested.
>
> **Note on S3 Versioning**: This is *different* than S3 Object Versioning. CloudFront always requests the current version of an S3 object. To use this strategy with S3, you must physically change the filename of the object in the bucket and update your application's HTML/Code to point to the new filename.
