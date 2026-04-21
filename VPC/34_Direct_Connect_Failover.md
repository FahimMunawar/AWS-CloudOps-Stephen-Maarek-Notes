# 34 — Direct Connect + Site-to-Site VPN Failover

## Architecture

Use Direct Connect as the **primary** connection and Site-to-Site VPN as the **backup**:

```
Corporate Data Center
  │
  ├── Primary:  Direct Connect (private, fast, expensive)
  │
  └── Backup:   Site-to-Site VPN (public internet, encrypted)
                → kicks in automatically if Direct Connect fails
  │
  ▼
AWS VPC
```

---

## Why Not Use a Second Direct Connect as Backup?

| Backup Option | Cost | Reliability |
|---|---|---|
| Second Direct Connect | High (expensive) | Very high |
| **Site-to-Site VPN** | Low | Good — public internet is widely available |

Site-to-Site VPN is a cost-effective failover because the public internet is generally accessible even when a dedicated Direct Connect link fails.

---

## SysOps Exam Q&A

**Q: A company uses Direct Connect as its primary connection to AWS. They want a cost-effective backup if Direct Connect fails. What do you recommend?**
A: Site-to-Site VPN as a backup — it uses the public internet (encrypted) and is much cheaper than provisioning a second Direct Connect connection.

---

## Quick Reference

```
DX + VPN failover pattern:
  Primary:  Direct Connect (private, low latency, expensive)
  Backup:   Site-to-Site VPN (public internet, encrypted, cheap)

Second DX as backup = expensive but maximum resiliency
VPN as backup = cost-effective, leverages public internet availability
```
