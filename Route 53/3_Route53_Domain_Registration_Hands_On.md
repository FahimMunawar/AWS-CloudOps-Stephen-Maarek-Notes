# 3 — Route 53: Domain Registration (Hands-On)

> **Cost**: Domain registration starts at ~$12–13/year. Skip registration and watch the section if you don't want to pay.

---

## Registering a Domain

**Route 53 → Register domains → Register domain**

1. Enter a unique domain name → check availability → add to cart
2. **Duration**: 1 year (minimum)
3. **Auto-renew**: turn OFF if only using for the course; ON if keeping long-term
4. **Contact information**: pre-populated from your AWS account
5. **Privacy protection**: enable — hides your personal contact info from public WHOIS lookups
6. Review → Accept terms → Submit (charged immediately)

> Registration can take a few minutes to a few hours to complete.

---

## After Registration: Hosted Zone

**Route 53 → Hosted zones → your-domain.com**

Two records are auto-created:

| Record | Purpose |
|---|---|
| **NS** (Name Server) | Points to Route 53 AWS DNS servers — Route 53 is authoritative for this domain |
| **SOA** (Start of Authority) | Administrative information about the zone |

> The NS record means Route 53 is the source of truth for all DNS records in this zone. Any records you create here will be served by Route 53 DNS servers.

---

## Quick Reference

```
Register domain: Route 53 → Register domains → search → checkout
  Cost: $12–13+/year
  Auto-renew: OFF for course use, ON for long-term
  Privacy protection: enable to hide personal contact info

After registration:
  Hosted zone auto-created with:
    NS record  → Route 53 is authoritative DNS
    SOA record → zone administrative info

Next step: add A, AAAA, CNAME records to the hosted zone
```
