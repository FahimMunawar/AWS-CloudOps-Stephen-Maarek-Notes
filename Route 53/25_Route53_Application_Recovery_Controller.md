# 25 — Route 53 Application Recovery Controller (ARC)

## Overview

Automates and orchestrates application recovery across AZs and regions for mission-critical applications requiring high availability and minimal downtime.

---

## Use Cases

- Multi-region **active/standby** or **active/active** setups
- Regulatory or business continuity requirements
- Mission-critical applications needing controlled failover

---

## Key Features

| Feature | Detail |
|---|---|
| **Readiness Checks** | Continuously validate that standby/replica infrastructure is ready to take over |
| **Routing Controls** | Traffic switches that reroute users between AZs/regions via DNS and health checks |
| **Zonal Shift** | Shift traffic away from an impaired AZ |
| **Region Shift** | Orchestrate a full region-level failover |

---

## Architecture Example

```
Active Region (us-east-1):          Standby Region (eu-west-2):
  ALB                                 ALB
  EC2 instances (multi-AZ)            EC2 instances (multi-AZ)
  DynamoDB                            DynamoDB

Route 53 Application Recovery Controller:
  Recovery Group
    ├── Resource Set: Load Balancers  → Readiness Check
    ├── Resource Set: EC2 instances   → Readiness Check
    └── Resource Set: DynamoDB        → Readiness Check

When readiness checks pass → controlled failover via Route 53 DNS
```

---

## Quick Reference

```
ARC: automate multi-region/multi-AZ failover for mission-critical apps

Components:
  Recovery Group → contains resource sets
  Resource Sets  → grouped resources (ALB, EC2, DynamoDB, etc.)
  Readiness Checks → validate standby is ready before failover
  Routing Controls → DNS-level traffic switches

Operations:
  Zonal Shift  → move traffic away from impaired AZ
  Region Shift → full region-level failover

Integration: Route 53 DNS + health checks orchestrate the actual traffic switch
```
