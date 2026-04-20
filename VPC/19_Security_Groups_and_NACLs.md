# 19 — Security Groups and Network ACLs (NACLs)

## Overview

Two layers of network security in a VPC:

| Layer | Resource | Scope |
|---|---|---|
| NACL | Subnet boundary | All EC2 instances in associated subnet |
| Security Group | EC2 instance | Specific instance(s) |

---

## Stateful vs Stateless — The Critical Difference

### Stateful (Security Group)
Return traffic is **automatically allowed** — outbound rules are not evaluated for accepted inbound connections, and vice versa.

### Stateless (NACL)
Every packet is evaluated independently — both inbound AND outbound rules are always checked.

---

## Inbound Request Flow

```
Internet
  ↓
NACL inbound rules evaluated          ← stateless: must explicitly allow
  ↓ (allowed)
Security Group inbound rules evaluated ← stateful: if allowed in...
  ↓ (allowed)
EC2 Instance processes request
  ↓
Security Group outbound rules         ← SKIPPED (stateful — auto-allowed)
  ↓
NACL outbound rules evaluated         ← stateless: must explicitly allow
  ↓
Internet
```

### Outbound Request Flow (EC2 → Internet)

```
EC2 Instance
  ↓
Security Group outbound rules evaluated
  ↓ (allowed)
NACL outbound rules evaluated
  ↓
Internet (e.g., google.com)
  ↓
NACL inbound rules evaluated          ← stateless: must allow return traffic
  ↓
Security Group inbound rules          ← SKIPPED (stateful — auto-allowed)
  ↓
EC2 Instance receives response
```

---

## NACL Rules

| Property | Detail |
|---|---|
| Rule numbers | 1 – 32,766 |
| Priority | Lower number = higher priority |
| First match wins | Rule 100 ALLOW beats rule 200 DENY for same IP |
| Last rule | `*` — implicit DENY ALL (no rule number) |
| Increment recommendation | Add rules in steps of 100 (leaves room to insert) |
| New NACL default | **Deny everything** |
| Default NACL default | **Allow everything** in and out |

> Best practice: do NOT modify the default NACL — create a custom one instead.

---

## Default NACL

The NACL auto-assigned to subnets **allows all inbound and outbound traffic**. This keeps new subnets functional without manual configuration.

---

## Ephemeral Ports

When a client connects to a server, the OS opens a random **ephemeral (short-lived) port** for the server to send the response back to.

| OS | Ephemeral Port Range |
|---|---|
| Windows 10 | 49,152 – 65,535 |
| Linux | 32,768 – 60,999 |

### Why This Matters for NACLs

NACLs are stateless — the return traffic must be explicitly allowed. Example: web EC2 → database EC2:

| NACL | Direction | Rule |
|---|---|---|
| Web NACL | Outbound | Allow TCP port 3306 → DB subnet CIDR |
| DB NACL | Inbound | Allow TCP port 3306 from Web subnet CIDR |
| DB NACL | Outbound | Allow TCP ports **1024–65,535** → Web subnet CIDR (ephemeral) |
| Web NACL | Inbound | Allow TCP ports **1024–65,535** from DB subnet CIDR (ephemeral) |

> Security groups don't need ephemeral port rules — they're stateful and auto-allow return traffic.

---

## Multi-Subnet NACL Consideration

Each NACL rule uses CIDR ranges. When adding new subnets, **update all NACL rules** to include the new subnet's CIDR — otherwise cross-subnet traffic will be blocked.

---

## Security Group vs NACL — Exam Comparison

| Factor | Security Group | NACL |
|---|---|---|
| Operates at | Instance level | Subnet level |
| Rule types | Allow only | Allow **and Deny** |
| Statefulness | **Stateful** (return traffic auto-allowed) | **Stateless** (all traffic evaluated) |
| Rule evaluation | All rules evaluated | First match wins (priority order) |
| Applies to | Specific instances | All instances in associated subnet |
| Block specific IP | Cannot deny | **Can deny** — key use case |

---

## SysOps Exam Q&A

**Q: How do you block a specific IP address from accessing your VPC?**
A: Use a NACL deny rule — security groups only support allow rules.

**Q: Why must you include ephemeral port ranges in NACL outbound rules?**
A: NACLs are stateless — the response from a server uses a random ephemeral port assigned by the client OS. Without allowing that range, responses are blocked.

**Q: What is the default behavior of a newly created custom NACL?**
A: Denies all inbound and outbound traffic (only the `*` deny-all rule exists).

**Q: A security group allows inbound HTTP. Does outbound HTTP need to be allowed for the response?**
A: No — security groups are stateful. The return traffic is automatically allowed.

**Q: NACL rule 100 allows `192.168.1.1/32` and rule 200 denies it. What happens?**
A: Traffic is allowed — rule 100 has higher priority (lower number wins, first match wins).

---

## Quick Reference

```
NACL (stateless, subnet level):
  Rules 1–32,766; lower = higher priority; first match wins; * = implicit deny
  Allow + Deny rules; new custom NACL = deny all; default NACL = allow all
  Must allow ephemeral ports (1024–65,535) for return traffic
  Best use: block specific IP addresses

Security Group (stateful, instance level):
  Allow rules only; all rules evaluated; return traffic auto-allowed
  No ephemeral port concern

Key exam distinction:
  Need to DENY a specific IP → use NACL
  Stateful return traffic → Security Group handles automatically
  Stateless → NACL requires explicit inbound + outbound rules
```
