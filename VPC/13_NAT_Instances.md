# 13 — NAT Instances

## Overview

NAT (Network Address Translation) instances allow EC2 instances in **private subnets** to initiate outbound internet traffic by routing through an EC2 instance in a **public subnet**.

> **Outdated** — NAT Gateway is the recommended replacement. NAT instances still appear on the exam as a comparison target.

---

## How NAT Works (Packet Rewriting)

```
Private EC2 (10.0.0.20)
  │  src: 10.0.0.20  dst: 50.60.4.10
  ▼
NAT Instance (public subnet, Elastic IP: 12.34.56.78)
  │  rewrites src IP → 12.34.56.78
  │  src: 12.34.56.78  dst: 50.60.4.10
  ▼
Public Server (50.60.4.10)
  │  reply: src: 50.60.4.10  dst: 12.34.56.78
  ▼
NAT Instance
  │  translates back → forwards to 10.0.0.20
  ▼
Private EC2
```

The NAT instance **rewrites the source IP** — this is why source/destination check must be disabled.

---

## Setup Requirements

| Requirement | Detail |
|---|---|
| **Placement** | Must be in a **public subnet** |
| **Elastic IP** | Must have a fixed Elastic IP attached |
| **Source/Destination Check** | Must be **disabled** on the NAT instance |
| **Route Table** | Private subnet route: `0.0.0.0/0 → NAT instance` |
| **AMI** | Pre-configured Amazon Linux NAT AMI (end of support: Dec 31, 2020) |

---

## Security Group Rules

| Direction | Allow |
|---|---|
| Inbound | HTTP/HTTPS from private subnet CIDRs; SSH from home/admin IP |
| Outbound | HTTP/HTTPS to internet |

---

## Limitations vs NAT Gateway

| Factor | NAT Instance | NAT Gateway |
|---|---|---|
| Availability | Not HA — manual multi-AZ setup needed | Managed, HA within AZ |
| Bandwidth | Limited by instance size | Up to 100 Gbps |
| Management | Must manage SGs, patching, failover | Fully managed by AWS |
| Status | End of standard support (Dec 2020) | Recommended |

---

## SysOps Exam Q&A

**Q: Why must source/destination check be disabled on a NAT instance?**
A: NAT instances rewrite packet source IPs — EC2's default check drops packets where the instance is not the source or destination, which would break NAT functionality.

**Q: What does a NAT instance require to function?**
A: Public subnet placement, an Elastic IP, source/destination check disabled, and a private subnet route pointing `0.0.0.0/0` to the NAT instance.

**Q: When would you choose a NAT instance over a NAT Gateway?**
A: NAT instances can be used as a bastion host simultaneously and are cheaper for very low traffic — but NAT Gateway is preferred in all production scenarios.

**Q: What happens if the NAT instance fails?**
A: Internet access for all private subnets depending on it is lost — no automatic failover without manual setup (ASG across AZs).

---

## Quick Reference

```
NAT Instance: outdated but exam-relevant
  Must be in public subnet + Elastic IP
  Disable source/destination check (rewrites packet src IP)
  Private subnet route: 0.0.0.0/0 → NAT instance

Limitations: not HA, bandwidth = instance size, manual SG management
NAT Gateway: managed, HA, no source/dest check needed → preferred
```
