# 23 — Route 53: DNS Query Logging

## Two Types of Logging

| Type | Scope | Destination |
|---|---|---|
| **DNS Query Logging** | Public hosted zones only | CloudWatch Logs → S3 (export) |
| **Resolver Query Logging** | Resources within VPC (private) | S3, CloudWatch Logs, or Kinesis Data Firehose |

---

## DNS Query Logging (Public Hosted Zones)

Logs all public DNS queries received by Route 53 Resolver.

Log fields:
- Log format version, timestamp
- Hosted zone ID
- Query name, query type
- Response code, protocol
- Edge location, Resolver IP address
- EDNS client subnet

---

## Resolver Query Logging (VPC Resources)

Logs DNS queries from resources inside your VPC, including:
- Private hosted zone queries
- Resolver inbound/outbound endpoint queries
- Resolver DNS Firewall queries

Output format: JSON document (account ID, version, query details, etc.)

### Sharing

Resolver query logging configuration can be **shared across AWS accounts** using **AWS Resource Access Manager (RAM)**.

---

## Quick Reference

```
DNS Query Logging:
  Scope: public hosted zones
  Destination: CloudWatch Logs (→ export to S3)

Resolver Query Logging:
  Scope: VPC resources (private hosted zones, resolver endpoints, DNS Firewall)
  Destination: S3 / CloudWatch Logs / Kinesis Data Firehose
  Sharing: via AWS RAM (cross-account)
```
