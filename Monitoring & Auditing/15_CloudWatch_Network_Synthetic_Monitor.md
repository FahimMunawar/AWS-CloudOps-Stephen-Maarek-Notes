# 15 — CloudWatch Network Synthetic Monitor

## Overview

Monitors network performance between your on-premises data center and AWS over Direct Connect or Site-to-Site VPN — no agents required.

---

## What It Detects

- Packet loss
- Latency
- Jitter

---

## Key Details

| Detail | Value |
|---|---|
| **Connections supported** | AWS Direct Connect, Site-to-Site VPN |
| **Traffic tested** | ICMP or TCP on IPv4 |
| **Agent required** | No |
| **Metrics output** | CloudWatch Metrics (real-time) |

---

## Quick Reference

```
Network Synthetic Monitor = agentless network health monitoring for hybrid connectivity
Tests: ICMP / TCP (IPv4) over Direct Connect or Site-to-Site VPN
Detects: packet loss, latency, jitter
Output: CloudWatch Metrics
Use case: verify on-premises ↔ AWS connection is performing as expected
```
