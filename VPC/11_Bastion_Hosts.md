# 11 — Bastion Hosts

## Overview

A Bastion Host is an EC2 instance in a **public subnet** used as a jump server to SSH into EC2 instances in **private subnets**.

---

## Architecture

```
Internet (User's computer)
  │
  │  SSH (port 22)
  ▼
Bastion Host EC2 (public subnet)
  │  Security Group: Bastion-SG
  │
  │  SSH (port 22)
  ▼
Private EC2 Instance(s) (private subnet)
  │  Security Group: Private-SG
```

---

## Security Group Rules (Exam Focus)

| Resource | Security Group Rule | Recommended |
|---|---|---|
| **Bastion Host** | Allow inbound SSH (port 22) | Restrict to corporate/user public IPs only — NOT `0.0.0.0/0` |
| **Private EC2** | Allow inbound SSH (port 22) | Source = Bastion Host **private IP** or **Bastion-SG** |

> Restricting the bastion host SG to known IP ranges minimizes attack surface — an attacker who gains bastion access can pivot to all private instances.

---

## Key Points

- Bastion host **must be in a public subnet** (needs IGW + route table)
- One bastion host can reach **many private EC2 instances**
- Private EC2 security group references the **bastion host's private IP** (not public) — traffic originates from inside the VPC
- Referencing the **security group** of the bastion (instead of its IP) is equivalent and more flexible

---

## SysOps Exam Q&A

**Q: A developer needs SSH access to a private EC2 instance. What is the recommended approach?**
A: Deploy a bastion host in a public subnet. The developer SSHs to the bastion first, then from the bastion to the private EC2.

**Q: What should the bastion host's security group allow for inbound SSH?**
A: Only the specific corporate/user public IP CIDR — never `0.0.0.0/0`.

**Q: What should the private EC2's security group allow for SSH?**
A: Inbound port 22 from the bastion host's private IP or from the bastion host's security group.

**Q: Why does the private EC2 reference the bastion's private IP (not public IP) in its security group?**
A: SSH traffic from the bastion to the private EC2 travels within the VPC — it originates from the bastion's private IP.

---

## Quick Reference

```
Bastion Host = jump server in public subnet → SSH into private EC2s

Bastion SG:   inbound SSH (22) from corporate/user IPs only (NOT 0.0.0.0/0)
Private SG:   inbound SSH (22) from bastion private IP or bastion SG

Flow: User → SSH → Bastion (public subnet) → SSH → Private EC2 (private subnet)

Security principle: restrict bastion SG as tightly as possible
```
