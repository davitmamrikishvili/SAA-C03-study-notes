---
tags:
  - aws/api-gateway
  - networking
  - serverless
category: Networking
---

# 🌐 API Gateway

> [!INFO] Definition
> **Amazon API Gateway** is a fully managed service for creating, publishing, and managing APIs at any scale. It acts as the **front-door** (entry-point) for applications, sitting between clients and backend integrations (Lambda, HTTP endpoints, AWS services).

## 🏗️ Core Characteristics
* **Public Service**: Accessible via public endpoints. Can connect to services in **AWS** or **on-premises**.
* **Highly Available & Scalable**: Managed by AWS — no infrastructure to provision.
* **Built-in Features**: Authorization, throttling, caching, CORS, request/response transformations, OpenAPI spec import/export, and direct AWS service integrations.
* **API Types**:
	* **REST APIs**: Full-featured, supports API keys, per-client throttling, request validation.
	* **HTTP APIs**: Simpler, lower-cost, lower-latency alternative to REST APIs.
	* **WebSocket APIs**: For real-time, two-way communication (e.g., chat apps, live dashboards).

![[APIGateway-1.png]]

---

## 🔐 Authentication

* API Gateway supports a range of authentication methods, or APIs can be left **completely open** (unauthenticated).
* **Amazon Cognito User Pools**: Clients authenticate with Cognito and receive a token. API Gateway validates the token before forwarding the request to the backend.
* **Lambda Authorizers** (Custom): A Lambda function evaluates the request (e.g., bearer token, request parameters) and returns an IAM policy.
* **IAM Authorization**: Requests are signed with AWS Signature v4 — ideal for service-to-service communication.

![[APIGateway-2.png]]

---

## 🌍 Endpoint Types

| Type               | Behavior                                                                                                                    |
| :----------------- | :-------------------------------------------------------------------------------------------------------------------------- |
| **Edge-Optimized** | Requests are routed to the nearest **CloudFront POP** before reaching the API. Best for geographically distributed clients. |
| **Regional**       | Optimized for clients in the **same region** as the API.                                                                    |
| **Private**        | Accessible **only within a VPC** via an **Interface VPC Endpoint**. No public exposure.                                     |

---

## 🚀 Stages & Deployments

* APIs are **deployed to stages** (e.g., `dev`, `staging`, `prod`). Each stage represents a snapshot of your API configuration.
* When you update an API configuration, you **deploy it to a stage** to make the changes live.
* Each stage gets its own **invoke URL** (e.g., `https://api-id.execute-api.region.amazonaws.com/prod`).

![[APIGateway-3.png]]

---

## ❌ Error Codes

Understanding API Gateway error codes is critical for the exam:

### 4XX — Client Errors
| Code    | Meaning                                                                   |
| :------ | :------------------------------------------------------------------------ |
| **400** | **Bad Request** — generic client-side error (malformed syntax).           |
| **403** | **Access Denied** — authorizer denied the request, or blocked by **WAF**. |
| **429** | **Too Many Requests** — client has exceeded the **throttling** limit.     |

### 5XX — Server Errors
| Code    | Meaning                                                                                 |
| :------ | :-------------------------------------------------------------------------------------- |
| **502** | **Bad Gateway** — bad output returned by the backend (e.g., malformed Lambda response). |
| **503** | **Service Unavailable** — backend endpoint is offline or experiencing major issues.     |
| **504** | **Integration Timeout** — backend did not respond within the **29-second** hard limit.  |

> [!TIP] Exam Pointer: Error Code Triggers
> * **"Lambda returns malformed response"** → **502 Bad Gateway**.
> * **"API call times out"** → **504** (remember the **29-second** API Gateway integration timeout, distinct from Lambda's 15-minute limit).
> * **"Rate limit exceeded"** → **429 Too Many Requests**.

---

## 💾 Caching

* Caching is configured **per stage** to reduce the number of calls to the backend.
* Cache can be **encrypted** for sensitive data.
* **TTL**: Default is **300 seconds** (5 min). Configurable from 0 to 3600 seconds.

![[APIGateway-4.png]]
