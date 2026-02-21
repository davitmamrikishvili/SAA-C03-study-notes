---
tags:
  - aws/route53
  - networking
  - dns
category: Networking & Content Delivery
---

# DNS - Domain Name System

> [!ABSTRACT] How DNS Works
> DNS translates human-readable domain names (e.g., `amazon.com`) into machine-readable IP addresses (e.g., `192.0.2.1`).

## Core Components
* **DNS Client**: Your device (phone, laptop) making the request.
* **Resolver**: Usually your ISP or a public resolver (like Google 8.8.8.8) that queries DNS on your behalf.
* **Nameserver**: A server that hosts **Zonefiles**.
* **Zone**: A part of the DNS namespace (e.g., `google.com`).
* **Root Hints**: A file containing the IP addresses of the 13 logical root servers.
* **TLD (Top-Level Domain)**: `.com`, `.net`, `.org` (gTLD) or `.ge`, `.uk` (ccTLD).

## AWS Implementation: Route 53
* Route 53 is a **Global Service**.
* It provides both Domain Registration and Authoritative DNS.

## Common Record Types (Key for Exam)
* **A Record**: Maps name to IPv4 address.
* **AAAA Record**: Maps name to IPv6 address.
* **CNAME**: Maps name to another name (cannot be at the apex).
* **Alias**: AWS-specific; maps name to AWS resource (can be at the apex).