# 17 — Route 53: Traffic Flow

## Overview

A visual editor for building complex routing decision trees — instead of creating records one by one.

---

## Key Features

| Feature | Detail |
|---|---|
| **Visual UI** | Drag-and-drop routing policy builder |
| **Traffic Policy** | Saved, versioned routing configuration |
| **Versioning** | Edit and deploy as a new version |
| **Reusable** | Apply same policy to multiple hosted zones |
| **Supports all policies** | Weighted, Failover, Geolocation, Latency, Multivalue, Geoproximity |

---

## Traffic Policy Structure

```
Starting point: record type (A, AAAA, CNAME)
  └── Connect to a rule (Weighted / Failover / Geoproximity / etc.)
        └── Each rule branch → endpoint (IP or domain)
              └── (Optional) further nested rules
```

---

## Geoproximity with Traffic Flow

- Select **Geoproximity rule** → add locations (AWS regions or custom lat/lon)
- **Show Map** — visual preview of which geographic areas route to each endpoint
- Adjust **bias** and see the dividing line shift in real time on the map
- Add more regions to split the world further

```
Bias = 0: equal split by proximity
Bias > 0: expand that region's coverage (attract more users)
Bias < 0: shrink coverage (attract fewer users)
```

---

## Deploying a Traffic Policy

1. Create Traffic Policy → design the routing tree → Create
2. **Create Policy Record** — deploy to a hosted zone:
   - Hosted zone: your domain
   - Record name: e.g., `proximity.yourdomain.com`
   - TTL: set as desired

> **Cost: $50/month per policy record** — delete when done if testing.

---

## Editing

- Traffic policy records in the hosted zone show as **routing to a traffic policy record**
- Edit by clicking the record → goes directly into the Traffic Flow visual editor
- Save changes as a **new version** of the policy

---

## Quick Reference

```
Traffic Flow: visual editor for complex routing trees
  Supports: all routing policy types
  Saves as: Traffic Policy (versioned, reusable)
  Deploy as: Policy Record → hosted zone

Geoproximity in Traffic Flow:
  Visual map shows coverage per region
  Adjust bias → map updates in real time

Cost: $50/month per policy record (prorated)
Edit: click record in hosted zone → opens Traffic Flow editor
```
