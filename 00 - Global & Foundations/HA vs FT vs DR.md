---
tags:
  - aws/foundations
  - architecture
category: Cloud Concepts
---

# HA vs FT vs DR

> [!ABSTRACT] Comparison
> These three concepts define how a system handles failures, ranging from minor glitches to major disasters.

## High-Availability (HA)
* Aims to ensure an agreed level of operational performance, usually uptime, for a higher than normal period.
* Generally implemented using **multi-AZ** deployments.
* Goal: Minimize any outages.

## Fault-Tolerance (FT)
* The property that enables a system to continue operating even in the event of the failure of some of its components.
* FT is **more expensive** and harder to implement than HA.
* Zero downtime or data loss.

## Disaster Recovery (DR)
* A set of policies, tools, and procedures to enable the recovery or continuation of vital technology infrastructure and systems following a natural or human-induced disaster.
* Used when HA and FT fail.

## Summary Table
| Concept | Goal | Implementation Cost |
| --- | --- | --- |
| **HA** | Minimize outages | Moderate |
| **FT** | Zero downtime | High |
| **DR** | Infrastructure recovery | Varies |