# 24 — Route 53 Resolver DNS Firewall

## Overview

A managed firewall that filters **outbound DNS requests** from resources within your VPC through the Route 53 Resolver.

---

## How It Works

```
EC2 instance → DNS query (example.com)
  → Route 53 Resolver
    → DNS Firewall
        ├── Blocked domain → returns no data (query blocked)
        └── Allowed domain → resolves normally
```

---

## Capabilities

| Capability | Detail |
|---|---|
| **Block malicious domains** | Prevent DNS queries to known bad domains |
| **Whitelist trusted domains** | Allow only approved domains |
| **Prevent DNS exfiltration** | Stop compromised applications from leaking data via DNS to attacker-controlled domains |

---

## Management & Logging

| Item | Detail |
|---|---|
| **Managed from** | AWS Firewall Manager |
| **Logs** | CloudWatch Logs + Route 53 Resolver Query Logs |

---

## Quick Reference

```
DNS Firewall: filters outbound DNS queries from VPC resources
  Block: malicious/untrusted domains
  Allow: whitelist trusted domains
  Attack prevented: DNS exfiltration (data leak via DNS)

Managed by: AWS Firewall Manager
Logs: CloudWatch Logs + Resolver Query Logs
```
