# 5 — Route 53: Lab Infrastructure Setup

## What Was Created

Three EC2 instances (one per region) + one ALB in Frankfurt — used as targets for Route 53 routing policy demos.

---

## EC2 Instances

| Region | Location Code | AZ (example) |
|---|---|---|
| Frankfurt | `eu-central-1` | eu-central-1b |
| N. Virginia | `us-east-1` | us-east-1a |
| Singapore | `ap-southeast-1` | ap-southeast-1b |

### Launch Settings (each instance)

- AMI: Amazon Linux 2
- Type: t2.micro
- Key pair: none (use EC2 Instance Connect if needed)
- Security group: allow SSH + HTTP from anywhere (0.0.0.0/0)
- User data: script that outputs `Hello World from <AZ>` — uses `EC2_AVAILABILITY_ZONE` env variable

### Verify

Access each instance via `http://<public-ip>` — response shows:
```
Hello World from <AZ name>
```

---

## ALB (Frankfurt)

| Setting | Value |
|---|---|
| Name | `DemoRoute53ALB` |
| Type | Application Load Balancer, internet-facing, IPv4 |
| Subnets | 3 AZ mappings |
| Security group | Same as EC2 (SSH + HTTP) |
| Target group | `demo-tg-route53` — instances type, port 80 |
| Target | Frankfurt EC2 instance |

Verify: access ALB DNS name via HTTP → returns `Hello World from eu-central-1b`

---

## Quick Reference

```
Lab setup: 3 EC2 instances (Frankfurt, N. Virginia, Singapore) + 1 ALB (Frankfurt)

Each EC2: Amazon Linux 2, t2.micro, HTTP open, user data → Hello World from <AZ>
ALB: points to Frankfurt EC2, target group on port 80

Keep public IPs + ALB DNS name handy for routing policy labs
```
