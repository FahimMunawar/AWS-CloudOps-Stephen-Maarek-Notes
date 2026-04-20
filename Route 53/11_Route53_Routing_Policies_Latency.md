# 11 — Route 53: Routing Policies — Latency-Based

## How It Works

Route 53 redirects users to the resource with the **lowest latency** — based on how quickly users can connect to each AWS region.

```
Users in Europe  → lowest latency to eu-central-1 → redirected there
Users in US      → lowest latency to us-east-1     → redirected there
Users in Asia    → lowest latency to ap-southeast-1 → redirected there
```

> Latency is evaluated against AWS regions, not geographic distance alone.

---

## Key Rules

| Rule | Detail |
|---|---|
| **Same DNS name + type** | All records in the set share the same name and type |
| **Region must be specified** | When using IP values (not Alias), you must manually specify which AWS region the IP belongs to |
| **Health checks** | Supported — can be combined |
| **Record ID** | Each record needs a unique identifier |

---

## Setup (per record)

- Record name: same for all (e.g., `latency.yourdomain.com`)
- Value: IP of the resource
- Routing policy: Latency
- **Region**: manually specify which AWS region this IP is in (e.g., `ap-southeast-1`)
- Record ID: descriptive name (e.g., `ap-southeast-1`)

---

## Quick Reference

```
Latency routing: route to AWS region with lowest latency for the user
  Use case: global apps where latency matters

Rules:
  Same DNS name + type across all records
  Specify AWS region per record (when using raw IPs)
  Supports health checks

Behavior:
  Each user gets the record pointing to the lowest-latency AWS region
  Same user always gets same answer unless latency changes or DNS cache expires
```
