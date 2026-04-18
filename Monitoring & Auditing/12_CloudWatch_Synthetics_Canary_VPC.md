# 12 — CloudWatch Synthetics Canary: VPC Deployment

## Overview

Synthetics Canary can be deployed inside a VPC to monitor private resources. Reporting results to CloudWatch requires either a NAT Gateway (public path) or a VPC endpoint (private path).

---

## Prerequisites

- **DNS Resolution** enabled in the VPC
- **DNS Hostnames** enabled in the VPC

---

## Option 1 — Report via NAT Gateway (Public Path)

```
Private Subnet
  └── EC2 instance  ←  Synthetics Canary monitors it
        │
        Canary reports results
        │
        ▼
    NAT Gateway (in public subnet)
        │
        ▼
    Internet → CloudWatch
```

- Traffic exits the VPC through NAT Gateway to reach CloudWatch publicly
- Simpler setup, but traffic traverses the public internet

---

## Option 2 — Report via VPC Endpoint (Private Path)

```
Private Subnet
  └── EC2 instance  ←  Synthetics Canary monitors it
        │
        Canary reports results
        │
        ▼
    VPC Endpoint for CloudWatch (Interface endpoint)
        │
        ▼
    CloudWatch (private, stays within AWS network)
```

- No traffic leaves AWS
- Also create additional endpoints as needed (e.g., **Gateway endpoint for S3**)

---

## SysOps Exam Q&A

**Q: What VPC settings must be enabled before deploying a Synthetics Canary inside a VPC?**
A: **DNS Resolution** and **DNS Hostnames** must both be enabled in the VPC.

**Q: A Synthetics Canary is in a private subnet and must report to CloudWatch without using the internet. What do you need?**
A: A **VPC Interface Endpoint for CloudWatch** — keeps all traffic within the AWS network.

---

## Quick Reference

```
Prerequisites: DNS Resolution + DNS Hostnames enabled in VPC

Public path:  Canary → NAT Gateway → Internet → CloudWatch
Private path: Canary → VPC Endpoint (CloudWatch) → CloudWatch

Also add: Gateway endpoint for S3 if S3 access needed privately
```
