# 1 — DNS Fundamentals

## What is DNS?

Domain Name System — translates human-friendly hostnames into IP addresses.

```
www.google.com → 142.250.x.x  (your browser uses the IP, not the name)
```

---

## DNS Hierarchy

```
.                        ← Root
└── .com                 ← Top Level Domain (TLD)
    └── example.com      ← Second Level Domain
        └── www.example.com       ← Subdomain
            └── api.www.example.com  ← FQDN (Fully Qualified Domain Name)
```

### Terminology

| Term | Example | Managed By |
|---|---|---|
| **TLD** (Top Level Domain) | `.com`, `.org`, `.gov` | ICANN |
| **Second Level Domain** | `amazon.com`, `google.com` | Domain registrar |
| **Subdomain** | `www.example.com` | You |
| **FQDN** | `api.www.example.com` | You |
| **Domain Registrar** | Route 53, GoDaddy | Third-party service |
| **Name Server** | Resolves DNS queries | Registrar / DNS provider |
| **Zone File** | Contains all DNS records | DNS provider |

---

## DNS Record Types (Preview)

A, AAAA, CNAME, NS — covered in detail in later lectures.

---

## How DNS Resolution Works

Example: browser wants to access `example.com` (IP: 9.10.11.12)

```
Browser → Local DNS Server (ISP or company-managed)
            ↓ (cache miss)
          Root DNS Server (ICANN)
            → "I know .com → NS record at 1.2.3.4"
            ↓
          TLD DNS Server (.com, managed by IANA) at 1.2.3.4
            → "I know example.com → NS record at 5.6.7.8"
            ↓
          Second-Level Domain DNS Server (domain registrar) at 5.6.7.8
            → "example.com = A record → 9.10.11.12"
            ↓
          Local DNS Server caches the answer
            ↓
          Browser → connects to 9.10.11.12
```

> Local DNS server caches the result — future queries for the same domain are answered immediately without traversing the hierarchy again.

---

## Quick Reference

```
DNS: hostname → IP translation
Hierarchy: Root → TLD (.com) → Second Level (example.com) → Subdomain

Resolution flow (recursive):
  Browser → Local DNS → Root NS → TLD NS → Domain NS → IP answer
  Local DNS caches result for future queries

Key components:
  Domain Registrar: where you register domain names (Route 53, GoDaddy)
  Name Server: resolves queries
  Zone File: contains DNS records
  A record: maps hostname to IPv4 address
```
