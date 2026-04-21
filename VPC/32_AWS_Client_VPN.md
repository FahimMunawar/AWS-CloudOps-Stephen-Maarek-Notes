# 32 — AWS Client VPN

## Overview

Allows your computer to privately connect to an AWS VPC (and optionally on-premises) over the public internet using OpenVPN.

```
Your Computer (OpenVPN client)
  │
  │  VPN tunnel (public internet, encrypted)
  │
  ▼
AWS Client VPN Endpoint (VPC)
  │
  ├── Private EC2 instances (accessed via private IP)
  └── On-premises network (via Site-to-Site VPN)
```

---

## Key Points

| Property | Detail |
|---|---|
| Protocol | OpenVPN |
| Transport | Public internet (encrypted) |
| Access | As if you are physically inside the VPC private network |
| EC2 access | Use **private IP** directly — no bastion host needed |
| On-premises | If VPC has a Site-to-Site VPN, client VPN extends access to on-premises too |

---

## Use Case

Access EC2 instances in a private subnet using their private IPs from your laptop — without needing a bastion host or making subnets public.

---

## Site-to-Site VPN Extension

```
Your Computer
  → Client VPN → VPC
                  → Site-to-Site VPN → On-Premises Data Center
```

Once connected via Client VPN, your machine can also reach on-premises servers privately through the existing VPN connection.

---

## SysOps Exam Q&A

**Q: A developer needs to access an EC2 instance in a private subnet using its private IP from their laptop. What is the simplest AWS solution?**
A: AWS Client VPN — the developer installs OpenVPN, connects to the Client VPN endpoint, and accesses the instance via private IP as if on the VPC network.

**Q: What protocol does AWS Client VPN use?**
A: OpenVPN.

---

## Quick Reference

```
AWS Client VPN:
  Your computer → OpenVPN → Client VPN endpoint → VPC private network
  Access EC2 via private IP directly (no bastion needed)
  Traffic: over public internet (encrypted)

Extended access: if VPC has Site-to-Site VPN → on-premises also reachable
```
