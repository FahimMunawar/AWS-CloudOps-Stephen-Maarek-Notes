# 20 — Domain Registrar vs DNS Service

## Key Distinction

| Concept | Purpose | Examples |
|---|---|---|
| **Domain Registrar** | Where you purchase/register a domain name | Route 53, GoDaddy, Google Domains |
| **DNS Service** | Where you manage DNS records (A, CNAME, etc.) | Route 53, GoDaddy DNS, Cloudflare |

> Every domain registrar typically includes a DNS service — but you are free to use a **different** DNS service from a different provider.

---

## Mixing Registrar and DNS Provider

**Scenario**: Domain registered with GoDaddy, DNS managed in Route 53.

```
1. Purchase domain at GoDaddy (example.com)
2. Create a Public Hosted Zone in Route 53 for example.com
3. Note the 4 NS (name server) records from the Route 53 hosted zone
4. Go to GoDaddy → domain settings → Custom Name Servers
5. Replace GoDaddy's name servers with the 4 Route 53 name servers
   → DNS queries for example.com now go to Route 53
   → All records managed in Route 53
```

---

## Why Use This Pattern?

- Route 53 has advanced features (health checks, geoproximity, traffic flow) not available in most registrar DNS services
- Centralize DNS management in Route 53 even when domains are bought elsewhere

---

## Quick Reference

```
Domain registrar ≠ DNS service (though registrars usually include basic DNS)

Use Route 53 as DNS with a third-party registrar:
  1. Create public hosted zone in Route 53
  2. Get the 4 NS records from the hosted zone
  3. Update NS records at the registrar → point to Route 53 name servers
  → Route 53 becomes authoritative DNS for your domain
```
