# 40 — VPC Traffic Mirroring

## Overview

Captures and inspects network traffic from ENIs in a **non-intrusive** manner — the source instance is unaware its traffic is being mirrored.

---

## How It Works

```
EC2 Instance (Source A)
  │  ENI  ← traffic mirrored (source unaware)
  │
  ├──→ Normal traffic flow continues (internet / other resources)
  │
  └──→ [VPC Traffic Mirroring] ──→ Network Load Balancer
                                          │
                                   ASG of EC2 instances
                                   (security software / analyzers)
```

- **Source:** one or more ENIs to capture traffic from
- **Target:** another ENI or a Network Load Balancer
- **Filter:** optional — capture only specific traffic (e.g., specific ports, protocols)

---

## Key Properties

| Property | Detail |
|---|---|
| Non-intrusive | Source instance continues functioning normally |
| Multiple sources | Mirror multiple ENIs to the same target (NLB) |
| Scope | Same VPC, or across VPCs with VPC Peering enabled |
| Filter | Optional — capture subset of traffic |

---

## Use Cases

- **Content inspection** — analyze packet payloads
- **Threat monitoring** — detect anomalous traffic patterns
- **Network troubleshooting** — deep packet inspection without disrupting workloads

---

## SysOps Exam Q&A

**Q: You need to inspect all network traffic on EC2 instances without disrupting their operation. What feature do you use?**
A: VPC Traffic Mirroring — mirrors ENI traffic to a Network Load Balancer (backed by security appliances) without affecting the source instances.

**Q: Can VPC Traffic Mirroring work across VPCs?**
A: Yes — source and target can be in different VPCs if VPC Peering is enabled between them.

---

## Quick Reference

```
VPC Traffic Mirroring:
  Source: ENI(s) to capture from
  Target: ENI or Network Load Balancer
  Filter: optional — capture specific traffic only
  Non-intrusive: source instance unaffected

Scope: same VPC or across VPCs (requires VPC Peering)
Use cases: content inspection, threat monitoring, network troubleshooting
```
