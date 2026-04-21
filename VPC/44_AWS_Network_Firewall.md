# 44 — AWS Network Firewall

## Overview

A managed firewall that protects an **entire VPC** — Layer 3 to Layer 7 inspection in all traffic directions.

---

## Network Protection Comparison (SysOps Summary)

| Service | Protects | Against |
|---|---|---|
| Security Groups | EC2 instance level | Unauthorized access |
| NACLs | Subnet level | Unauthorized access |
| AWS WAF | HTTP/HTTPS requests | Malicious web requests |
| AWS Shield / Shield Advanced | AWS services | DDoS attacks |
| AWS Firewall Manager | Multi-account | Centrally manages WAF + Shield rules |
| **AWS Network Firewall** | **Entire VPC (L3–L7)** | **All network threats** |

---

## Traffic Directions Covered

Network Firewall inspects traffic in all directions:

- VPC ↔ Internet (inbound and outbound)
- VPC ↔ Peered VPC
- VPC ↔ Direct Connect
- VPC ↔ Site-to-Site VPN

---

## Under the Hood

Internally uses **AWS Gateway Load Balancer** — but AWS manages the appliances, so you only define rules without setting up third-party appliances.

Multi-account rule management: **AWS Firewall Manager**

---

## Rule Capabilities

| Rule Type | Example |
|---|---|
| IP / Port filtering | Block tens of thousands of IPs |
| Protocol filtering | Disable outbound SMB protocol |
| Domain-level filtering | Allow outbound only to `mycorp.com` or approved software repos |
| Pattern matching | Regex-based traffic matching |
| Actions | Allow / Drop / Alert |
| Active flow inspection | Intrusion prevention system (IPS) capability |

---

## Log Destinations

Rule match logs can be sent to:
- Amazon S3
- CloudWatch Logs
- Kinesis Data Firehose

---

## SysOps Exam Q&A

**Q: You need to protect an entire VPC with a managed firewall that inspects traffic at Layers 3–7 in all directions. What do you use?**
A: AWS Network Firewall — it provides VPC-level protection for all inbound, outbound, and lateral traffic.

**Q: You want to block all outbound SMB traffic from your VPC. What service allows protocol-level filtering at the VPC level?**
A: AWS Network Firewall — it supports protocol-based filtering rules.

**Q: How do you centrally manage AWS Network Firewall rules across multiple accounts?**
A: AWS Firewall Manager.

---

## Quick Reference

```
AWS Network Firewall: VPC-level managed firewall (L3–L7)
  All traffic directions: internet, peered VPC, Direct Connect, Site-to-Site VPN
  Internally uses Gateway Load Balancer (AWS-managed appliances)

Rules: IP/port, protocol, domain, regex → Allow / Drop / Alert
Active flow inspection: IPS capability

Logs: S3, CloudWatch Logs, Kinesis Firehose
Multi-account management: AWS Firewall Manager
```
