# 22 — Route 53 Resolver

## Default Behavior

Route 53 Resolver (built into every VPC) answers DNS queries for:
- Local EC2 instance domain names
- Private hosted zone records
- Public name server records

All within your AWS account — no cross-environment resolution by default.

---

## Hybrid DNS Problem

When you connect AWS to an on-premises data center (via VPN or Direct Connect), you need DNS to resolve **both ways**:

| Direction | Use Case |
|---|---|
| On-premises → AWS | On-premises server resolves an AWS private hosted zone domain |
| AWS → On-premises | EC2 instance resolves an on-premises internal domain |

---

## Solution: Resolver Endpoints

### Inbound Endpoint (On-Premises → AWS)

On-premises DNS resolvers forward queries into AWS:

```
On-premises server
  → On-premises DNS resolver
    → Resolver Inbound Endpoint (in VPC)
      → Route 53 Resolver
        → Private hosted zone → answer returned
```

### Outbound Endpoint (AWS → On-Premises)

EC2 instances forward queries out to on-premises:

```
EC2 instance
  → Route 53 Resolver
    → Resolver Outbound Endpoint (in VPC)
      → On-premises DNS resolver
        → On-premises internal domain → answer returned
```

---

## Requirements

- VPN or Direct Connect must be established first (network connectivity prerequisite)
- Inbound endpoint: allows on-premises to resolve AWS private hosted zone names
- Outbound endpoint: allows AWS resources to resolve on-premises domain names
- Both can be used together for full bidirectional hybrid DNS

---

## Quick Reference

```
Route 53 Resolver: answers DNS for AWS resources by default

Hybrid DNS (VPN/Direct Connect required):
  Inbound endpoint  → on-premises resolves AWS private hosted zone domains
  Outbound endpoint → EC2 resolves on-premises internal domain names

Use both for full bidirectional DNS resolution between AWS and on-premises
```
