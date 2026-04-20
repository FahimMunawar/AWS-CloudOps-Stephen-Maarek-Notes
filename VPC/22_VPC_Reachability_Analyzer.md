# 22 — VPC Reachability Analyzer

## Overview

A network diagnostics tool that analyzes connectivity between two endpoints in a VPC **without sending any actual traffic** — it models the network configuration and evaluates every hop.

---

## How It Works

```
Instance A (ENI) → Security Group → NACL → Route Table → NACL → Security Group → ENI → Instance B
       ↑                                                                                      ↑
  Analyzer checks every component along this path for misconfigurations
```

- Does **not** send packets — purely configuration analysis
- Checks: security groups, NACLs, route tables, ENIs, peering connections, etc.
- Result: **Reachable** or **Not reachable** + exact component causing the block

---

## Key Features

| Feature | Detail |
|---|---|
| Scope | Two endpoints within your VPC(s) |
| Method | Configuration analysis — no packets sent |
| Output | Reachable / Not reachable + specific blocking component identified |
| Cross-VPC | Supports endpoints in different VPCs |

---

## Use Cases

- Troubleshoot why Instance A cannot reach Instance B
- Verify network configuration matches intended design
- Identify misconfigured security groups, NACLs, or route tables without manual inspection

---

## SysOps Exam Q&A

**Q: An EC2 instance cannot reach another EC2 instance. You want to identify the exact misconfiguration without generating network traffic. What tool do you use?**
A: VPC Reachability Analyzer — it analyzes configuration across security groups, NACLs, and route tables and pinpoints the blocking component.

**Q: What is the key distinction of VPC Reachability Analyzer vs actually pinging an instance?**
A: Reachability Analyzer performs configuration-only analysis — no packets are sent. It tells you *why* connectivity fails based on rules, not packet loss.

---

## Quick Reference

```
VPC Reachability Analyzer:
  Network diagnostics — analyzes config, sends NO packets
  Checks: SGs, NACLs, route tables, ENIs, peering
  Output: reachable / not reachable + exact blocking component

Use cases:
  Troubleshoot EC2-to-EC2 connectivity
  Validate network config is correct as intended
```
