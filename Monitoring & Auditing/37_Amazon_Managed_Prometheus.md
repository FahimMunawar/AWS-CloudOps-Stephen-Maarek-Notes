# 37. Amazon Managed Service for Prometheus

## Overview

**Amazon Managed Service for Prometheus** is a serverless, Prometheus-compatible monitoring service for container metrics. AWS handles scaling, storage, and availability — you use the same Prometheus data model and PromQL query language.

---

## Key Properties

| Property | Detail |
|----------|--------|
| **Type** | Serverless, fully managed |
| **Compatibility** | Prometheus-compatible (same data model + PromQL) |
| **Target workloads** | Container metrics (EKS, Kubernetes) |
| **Scaling** | Automatic — ingestion, storage, and query operations |
| **Availability** | Data replicated across **3 Availability Zones** |
| **Retention** | Default **150 days**; maximum **3 years** |

---

## Supported Data Sources (Ingest From)

| Source | Type |
|--------|------|
| **Amazon EKS** | Managed Kubernetes |
| **Self-managed Kubernetes** | On EC2 or on-premises |
| **Prometheus server** | Open-source Prometheus |
| **AWS Distro for OpenTelemetry** | AWS-managed OTel collector |
| **OpenTelemetry** | Open-source OTel |

---

## Architecture

```
Metrics sources:
  Amazon EKS             → ┐
  Self-managed Kubernetes → ┤
  Prometheus server       → ┼──► Amazon Managed Prometheus
  AWS Distro OTel         → ┤         (stores up to 3 years)
  OpenTelemetry           → ┘              ↓
                                  Amazon Managed Grafana
                                  (PromQL dashboards)
```

---

## Key Query Language

| Term | Detail |
|------|--------|
| **PromQL** | Prometheus Query Language — used to filter and aggregate metrics |
| **Same as OSS** | Identical to open-source Prometheus — no relearning required |

---

## Typical Use Pattern

1. Ingest container metrics from EKS or Kubernetes into Managed Prometheus
2. Connect **Amazon Managed Grafana** as the visualization layer
3. Use **PromQL** to query and build dashboards

---

## Best Practices

✓ **Pair with Amazon Managed Grafana** — the standard combination for container observability  
✓ **Use for EKS and Kubernetes workloads** — purpose-built for container metrics  
✓ **Rely on built-in HA** — 3-AZ replication means no extra redundancy configuration needed  
✓ **Extend retention beyond 150 days if needed** — configurable up to 3 years  

---

## SysOps Exam Focus

**Q1: "You are running Amazon EKS and want to collect and store Prometheus metrics long-term with high availability and no infrastructure to manage. What should you use?"**
- A) Self-hosted Prometheus on EC2
- B) Amazon Managed Service for Prometheus — serverless, 3-AZ replication, stores metrics up to 3 years
- C) Amazon CloudWatch custom metrics
- D) Amazon OpenSearch
- **Answer: B** — Managed Prometheus is purpose-built for this: serverless, HA, long retention, EKS-native

**Q2: "What query language does Amazon Managed Service for Prometheus use?"**
- A) SQL
- B) PromQL — the same query language as open-source Prometheus
- C) KQL (Kibana Query Language)
- D) CloudWatch Metrics Insights SQL
- **Answer: B** — PromQL is the Prometheus query language; Managed Prometheus is fully compatible with it

**Q3: "What is the typical architecture for visualizing Prometheus metrics from EKS?"**
- A) EKS → CloudWatch → QuickSight
- B) EKS → Amazon Managed Prometheus → Amazon Managed Grafana (PromQL dashboards)
- C) EKS → Kinesis → OpenSearch Dashboards
- D) EKS → CloudTrail → CloudWatch Logs Insights
- **Answer: B** — Managed Prometheus stores the metrics; Managed Grafana queries them with PromQL and builds dashboards

---

## Quick Reference

```
Amazon Managed Service for Prometheus:
  Type: serverless, Prometheus-compatible
  Purpose: container metrics (EKS, Kubernetes)
  Scaling: automatic (ingest, store, query)
  HA: 3-AZ replication
  Retention: 150 days default → up to 3 years
  Query language: PromQL (same as open-source Prometheus)

  Ingest from: EKS, Kubernetes, Prometheus server,
               AWS Distro OTel, OpenTelemetry

  Visualize with: Amazon Managed Grafana

Exam tip:
  "Prometheus-compatible container metrics, serverless, long retention"
  → Amazon Managed Service for Prometheus
  Standard combo: Managed Prometheus + Managed Grafana
```

---

**File: 37_Amazon_Managed_Prometheus.md**
**Status: SysOps-focused, exam-ready, concise format**
