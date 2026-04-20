# 15 — Route 53: Routing Policies — Geolocation

## How It Works

Routes based on **where the user is physically located** — not latency.

Most specific matching location wins (country > continent > default).

```
User in Germany → matched to Europe continent record → eu-central-1
User in India   → matched to Asia continent record   → ap-southeast-1
User in US      → matched to United States record    → us-east-1
User in Mexico  → no match → Default record          → eu-central-1 (default)
```

---

## Location Granularity (most to least specific)

1. US state
2. Country (e.g., United States, India)
3. Continent (e.g., Asia, Europe)
4. **Default** — catches all unmatched locations

> **Always create a Default record** — users from unmatched locations get no response without it.

---

## Key Rules

| Rule | Detail |
|---|---|
| **Location options** | Continent, Country, US State, Default |
| **Most specific wins** | Country overrides Continent for users in that country |
| **Default required** | Handles all locations not covered by other records |
| **Health checks** | Supported |

---

## Geolocation vs Latency

| Feature | Geolocation | Latency |
|---|---|---|
| **Based on** | User's actual location | Network latency to AWS region |
| **Use case** | Content localization, geo-restrictions | Performance optimization |
| **Default record** | Required | Not needed |

---

## Use Cases

- Website localization (serve German content to German users)
- Content distribution restrictions (geo-blocking)
- Regional load balancing by geography

---

## Quick Reference

```
Geolocation: route by user location (NOT latency)
  Granularity: US state > country > continent > default
  Most specific match wins

Default record: REQUIRED — catches unmatched locations
Health checks: supported

vs Latency:
  Geolocation = WHERE the user is
  Latency     = HOW FAST they can reach the region
```
