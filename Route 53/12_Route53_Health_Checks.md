# 12 — Route 53: Health Checks

## Overview

Health checks enable **automated DNS failover** — Route 53 only routes traffic to healthy endpoints.

Primary use: public resources (with a workaround for private resources).

---

## Three Types of Health Checks

| Type | What It Monitors |
|---|---|
| **Endpoint** | A public endpoint (application, server, AWS resource) |
| **Calculated** | Multiple child health checks combined into one parent |
| **CloudWatch Alarm** | A CloudWatch Alarm — used for private resources |

Health check metrics are visible in **CloudWatch Metrics**.

---

## Type 1: Endpoint Health Checks

- ~**15 global health checkers** send requests to your public endpoint from around the world
- **Healthy threshold**: if **>18%** of health checkers report healthy → Route 53 considers endpoint healthy
- **Interval options**: 30 seconds (standard) or 10 seconds (fast health check — higher cost)
- **Protocols**: HTTP, HTTPS, TCP
- **Healthy status codes**: 2xx or 3xx
- **Text-based check**: health checkers can inspect first **5,120 bytes** of the response for specific text

### Network Requirement

Health checkers are on the public internet — your endpoint's security group/firewall must **allow inbound traffic from Route 53 health checker IP ranges**.

---

## Type 2: Calculated Health Checks

Combine multiple child health checks into one parent health check.

```
Parent health check
  ├── Child HC → EC2 instance 1
  ├── Child HC → EC2 instance 2
  └── Child HC → EC2 instance 3

Combine using: OR / AND / NOT
Specify: how many children must pass for parent to pass
Max children: 256
```

**Use case**: perform maintenance on one resource without triggering overall health check failure.

---

## Type 3: CloudWatch Alarm (Private Resources)

Route 53 health checkers are outside your VPC — they cannot reach private endpoints directly.

### Solution

```
Private EC2 / private resource
  → CloudWatch Metric (monitor health)
    → CloudWatch Alarm (triggers on breach)
      → Route 53 Health Check (monitors the alarm state)
        → Alarm state = ALARM → health check = Unhealthy
```

**Use case**: health checks for private VPC resources or on-premises resources.

---

## SysOps Exam Q&A

**Q: What percentage of Route 53 health checkers must report healthy for an endpoint to be considered healthy?**
A: **Greater than 18%** of the ~15 global health checkers.

**Q: How do you perform a health check on a private EC2 instance in a VPC?**
A: Create a **CloudWatch Metric + CloudWatch Alarm** on the instance, then create a Route 53 health check that monitors the **CloudWatch Alarm state**.

**Q: What is the minimum health check interval in Route 53?**
A: **10 seconds** (fast health check — higher cost). Standard is 30 seconds.

**Q: What is a Calculated Health Check?**
A: A parent health check that combines multiple child health checks using OR/AND/NOT logic. Supports up to 256 children.

---

## Quick Reference

```
Health check types:
  Endpoint        → public resource; ~15 global checkers; >18% healthy = healthy
  Calculated      → parent combines children (OR/AND/NOT); max 256 children
  CloudWatch Alarm → for private resources (VPC/on-premises)

Endpoint details:
  Interval: 30s (standard) or 10s (fast, costly)
  Protocols: HTTP, HTTPS, TCP
  Pass codes: 2xx or 3xx
  Text check: first 5,120 bytes of response
  Network: allow Route 53 IP ranges in your SG/firewall

Private resource pattern:
  CloudWatch Metric → CloudWatch Alarm → Route 53 HC monitors alarm state
```
