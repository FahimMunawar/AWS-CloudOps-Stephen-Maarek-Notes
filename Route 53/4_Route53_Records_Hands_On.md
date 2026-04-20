# 4 — Route 53: Creating Records (Hands-On)

## Creating a Record

**Route 53 → Hosted zones → your-domain.com → Create record**

| Field | Example | Notes |
|---|---|---|
| **Record name** | `test` | Creates `test.yourdomain.com` |
| **Record type** | A | IPv4 address |
| **Value** | `11.22.33.44` | The IP to route to |
| **TTL** | `300` | Seconds resolvers cache this record |
| **Routing policy** | Simple | Default; more options covered later |

---

## Verifying a Record from the Terminal

Install DNS tools on CloudShell (or any Linux terminal):
```bash
sudo yum install -y bind-utils
```

### nslookup
```bash
nslookup test.stephanetheteacher.com
# Returns: 11.22.33.44
```

### dig (preferred — shows TTL and record type)
```bash
dig test.stephanetheteacher.com
# Answer section: test.stephanetheteacher.com  A  11.22.33.44
# Also shows TTL value
```

> Windows: `nslookup` works natively. Mac: `dig` works natively.

---

## Quick Reference

```
Create record: Hosted zones → domain → Create record
  Name: subdomain prefix (e.g., "test" → test.yourdomain.com)
  Type: A (IPv4), AAAA (IPv6), CNAME, etc.
  Value: target IP or hostname
  TTL: cache duration in seconds (300 = 5 min default)

Verify:
  nslookup <domain>  → returns IP
  dig <domain>       → shows record type + TTL + IP (more detail)

CloudShell: install bind-utils first (sudo yum install -y bind-utils)
```
