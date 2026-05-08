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

---

## 🔒 CloudFront & SSL/TLS (ACM Integration)

Securing data in transit across a CDN requires careful coordination between CloudFront and your origin servers.

### 🛡️ AWS Certificate Manager (ACM)
ACM is a service that allows you to easily provision, manage, and deploy public and private Secure Sockets Layer/Transport Layer Security (SSL/TLS) certificates.
* **Supported Services Only**: ACM automatically handles certificate deployment and renewal for natively supported AWS services (e.g., CloudFront, Application Load Balancers, API Gateway).
> [!CAUTION] Crucial Exam Limitation
  > ACM **cannot** deploy certificates directly to EC2 instances. For custom origins like EC2 or on-premises servers, you must manually obtain, install, and manage a publicly trusted certificate.

### 🌐 CloudFront Default & Custom Domains
* **Default Domain**: Every Distribution receives a default `*.cloudfront.net` domain (e.g., `d111111abcdef8.cloudfront.net`). SSL is supported out-of-the-box for this domain.
* **Alternate Domain Names (CNAMEs)**: To use your own custom domain (e.g., `cdn.catagram.com`), you must:
  * Define the Alternate Domain Name on the Distribution.
  * Attach a matching, valid SSL/TLS certificate.
>[!WARNING] The `us-east-1` Rule
 > Because CloudFront is a global service, any certificate generated or imported into ACM for use with CloudFront **must be located in the `us-east-1` (N. Virginia) region**.

### 🔗 The Two Connections Architecture
When a user accesses CloudFront over HTTPS, there are actually **two distinct connections**, each requiring valid SSL configuration. Self-signed certificates will **not** work for either connection.

1. **Viewer Protocol (Viewer $\to$ CloudFront)**:
  * The viewer connects to the Edge Location.
  * The certificate applied to the Distribution must match the DNS name the customer used to request the content.
  * **SNI (Server Name Indication)**: Most modern browsers support SNI, allowing CloudFront to serve multiple SSL certificates from the same IP address. If you need to support legacy browsers without SNI, you must pay a significant premium (~$600/month) for a Dedicated IP Custom SSL.

2. **Origin Protocol (CloudFront $\to$ Origin)**:
  * The Edge Location connects to your Origin server.
  * The certificate installed on your Origin (ALB, EC2, etc.) must match the DNS name that CloudFront uses to contact it.
  * If the origin is an ALB, you can use ACM to manage its certificate (in the region the ALB resides). If it's an EC2 instance, you must manage it manually.


---

## 🔐 Securing Origins (Preventing Direct Bypass)

A common architectural requirement is ensuring that users *must* go through CloudFront to access your content, rather than bypassing the CDN and hitting your origin directly.

### 🛡️ Securing S3 Origins
S3 can act as an origin in two ways: as a raw S3 bucket (S3 Origin) or utilizing the Static Website Hosting feature (Custom Origin). To secure a true **S3 Origin**, you use an **OAI (Origin Access Identity)** or the newer **OAC (Origin Access Control)**.

* **What it is**: An OAI/OAC is a special CloudFront user identity associated with your distribution.
* **How it works**:
  1. CloudFront "becomes" this identity when requesting objects from S3.
  2. You configure your **S3 Bucket Policy** with a `Deny` for all public access, but an explicit `Allow` for the OAI/OAC.
  3. Result: The S3 bucket is totally isolated from the internet, accessible *only* via CloudFront.

### 🧱 Securing Custom Origins (EC2, ALB, On-Prem)
Because OAIs only work with S3, securing Custom Origins requires different techniques. You can use one or both of these approaches:

1. **Custom Headers**: Configure CloudFront to inject a secret custom header (e.g., `X-Shared-Secret: RandomString123`) before forwarding the request to the origin. Your origin (ALB/EC2) is configured to *reject* any request that lacks this header.
2. **IP Allowlisting**: AWS publishes the IP ranges of all CloudFront Edge Locations. You can configure your origin's Security Groups or traditional firewalls to *only* allow inbound traffic (port 80/443) from those specific CloudFront IP ranges, dropping all other direct public traffic.

---

## ⚡ Lambda@Edge

**Lambda@Edge** allows you to deploy and run lightweight Lambda functions directly across AWS's global network of CloudFront Edge Locations. It is used to inject custom compute logic directly into the HTTP request/response cycle.

### ⚠️ Limitations
* Currently supports **Node.js** and **Python** only.
* Runs in the AWS Public Space — it **cannot** access resources sitting privately inside your VPC.
* Does **not** support Lambda Layers.
* Subject to different (stricter) timeout and execution limits compared to standard Lambda.

### 🪝 The 4 Invocation Hooks

A Lambda@Edge function can be triggered at exactly four distinct phases of the CloudFront connection cycle:

![[CloudFrontLambdaAtEdge.png]]

1. **Viewer Request**: After CloudFront receives a request from a viewer, *before* it checks the cache.
2. **Origin Request**: *After* a cache miss, right before CloudFront forwards the request to the origin.
3. **Origin Response**: After CloudFront receives the response from the origin, *before* it caches it.
4. **Viewer Response**: Right before CloudFront returns the cached (or newly fetched) response to the viewer.

### 🎯 Common Use Cases
* **A/B Testing**: (Viewer Request) Inspecting cookies and silently rewriting the request URL to serve version A or version B to the user without changing the visible URL.
* **Device-Specific Routing**: (Origin Request) Inspecting the `User-Agent` header and fetching different objects depending on whether the user is on mobile or desktop.
* **Content by Country**: (Origin Request) Generating dynamic content based on the viewer's geolocation.
* **Origin Migration**: (Origin Request) Silently routing percentages of traffic between an old S3 bucket and a new one during a migration.

---

## 🔏 Private Distributions & Behaviours

By default, CloudFront distributions are **public** — any viewer can access any cached object via the distribution URL. For premium content, paid media, or sensitive resources, you need to restrict access so that only **authenticated and authorised users** can retrieve objects.

### 🌐 Public vs. Private Behaviours
* **Public Behaviour**: No access controls. Any viewer can request content. Standard use case for public websites, assets, and marketing content.
* **Private Behaviour**: Requests must include a **cryptographically signed** token. Without a valid signature, CloudFront returns a `403 Forbidden`. This is configured at the **Behaviour** level, meaning you can mix public and private behaviours within a single Distribution (e.g., `/free/*` is public, `/premium/*` is private).

### 🔑 Trusted Key Groups & Signers
To issue signed tokens, you need a **trusted key group** associated with the Distribution Behaviour.

* A **Key Group** contains one or more **public keys**.
* Your application holds the corresponding **private key** and uses it to cryptographically sign the URL or cookie before handing it to the user.
* CloudFront uses the public key in the Key Group to **verify** the signature on incoming requests.
* This is the **recommended** approach; the older method of using the AWS account's root CloudFront Key Pair is discouraged.

---

### 🎟️ Signed URLs vs. Signed Cookies

Both are mechanisms for granting access to private content, but they serve different purposes.

| Feature           | Signed URLs                                                         | Signed Cookies                                                                |
| :---------------- | :------------------------------------------------------------------ | :---------------------------------------------------------------------------- |
| **Scope**         | Access to a **single, specific object**.                            | Access to **multiple objects** (e.g., a whole subscription plan).             |
| **URL**           | The signature is embedded in the URL itself.                        | The signature is in a browser cookie. URL stays clean.                        |
| **Use Case**      | Individual file download, password-reset link, one-time attachment. | Paid content library, video streaming subscription, access to a set of files. |
| **Compatibility** | Works for all clients (no cookie support needed).                   | Requires a browser or client that supports cookies.                           |

> [!TIP] Exam PowerUP: Which to Use?
> * **Single file / one-time access** → **Signed URL**.
> * **Access to a group/subscription of objects without changing the URL** → **Signed Cookies**.
> * **Client doesn't support cookies** (e.g., custom HTTP client, CLI tool) → **Signed URL**.
