# 36. Amazon Managed Service for Grafana

## Overview

**Amazon Managed Service for Grafana** is a fully managed version of the open-source Grafana dashboarding tool. AWS handles provisioning, scaling, and maintenance so you can focus on building dashboards.

---

## What Is Grafana?

- Open-source tool for building dashboards and visualizing metrics, logs, and traces
- Widely used for operational monitoring and observability
- Hard to self-host at scale — AWS manages it for you

---

## Key Features

| Feature | Detail |
|---------|--------|
| **Fully managed** | AWS handles provisioning, scaling, and maintenance |
| **Workspaces** | Isolated Grafana environments — create one per team or use case |
| **Dashboards** | Visualize metrics, logs, and traces in one place |
| **Data sources** | Native AWS + third-party integrations |

---

## Supported Data Sources

### AWS Native
| Source | Data Type |
|--------|-----------|
| **Amazon CloudWatch** | Metrics and logs |
| **Amazon Managed Service for Prometheus** | Prometheus metrics |
| **AWS X-Ray** | Distributed traces |
| **Amazon OpenSearch** | Logs and search |

### Third-Party / Open Source
- Prometheus (self-hosted)
- Datadog
- Many other community data sources

---

## Architecture

```
Data Sources:
  CloudWatch → ┐
  Prometheus  → ┤
  X-Ray       → ┼──► Amazon Managed Grafana (workspace)
  OpenSearch  → ┤         ↓
  Datadog     → ┘    Dashboards (metrics, logs, traces)
```

---

## Best Practices

✓ **Use workspaces to isolate teams** — each workspace has its own users, dashboards, and data source configs  
✓ **Connect CloudWatch and X-Ray together** — correlate metrics and traces in a single Grafana dashboard  
✓ **Use managed service over self-hosted** — eliminates Grafana version management, storage, and HA concerns  

---

## SysOps Exam Focus

**Q1: "You want to build a unified dashboard that visualizes CloudWatch metrics, X-Ray traces, and Prometheus data in one place. Which AWS service should you use?"**
- A) Amazon QuickSight
- B) Amazon Managed Service for Grafana — integrates natively with CloudWatch, X-Ray, Prometheus, and OpenSearch
- C) AWS CloudWatch Container Insights
- D) Amazon Managed Service for Prometheus
- **Answer: B** — Grafana is the multi-source dashboarding tool; QuickSight is for BI/business data

**Q2: "What is the advantage of using Amazon Managed Service for Grafana over self-hosting Grafana?"**
- A) It provides more dashboard types than open-source Grafana
- B) AWS handles provisioning, scaling, and maintenance — you only manage dashboards and data sources
- C) It is the only way to visualize CloudWatch metrics
- D) It replaces the need for CloudWatch entirely
- **Answer: B** — The managed service eliminates operational overhead; dashboarding capability is the same as open-source Grafana

---

## Quick Reference

```
Amazon Managed Service for Grafana:
  Purpose: fully managed Grafana dashboarding
  AWS manages: provisioning, scaling, maintenance
  You manage: workspaces, dashboards, data sources

  Visualizes: metrics, logs, traces

  Native AWS sources:
    CloudWatch → metrics + logs
    Prometheus (managed) → Prometheus metrics
    X-Ray → distributed traces
    OpenSearch → logs

  Third-party: Prometheus (OSS), Datadog, and more

Exam signal: "unified dashboard across CloudWatch + X-Ray + Prometheus"
  → Amazon Managed Service for Grafana
```

---

**File: 36_Amazon_Managed_Grafana.md**
**Status: SysOps-focused, exam-ready, concise format**
