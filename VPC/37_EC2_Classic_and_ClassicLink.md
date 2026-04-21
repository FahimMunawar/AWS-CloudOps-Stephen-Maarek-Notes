# 37 — EC2-Classic and ClassicLink (Deprecated)

## Status: Deprecated — Likely Exam Distractors

| Feature | Status |
|---|---|
| EC2-Classic | Deprecated — cannot launch instances |
| ClassicLink | Deprecated |

---

## What They Were

**EC2-Classic:** AWS's original networking model — instances ran in a single flat network shared across all customers. Predates VPC.

**ClassicLink:** A feature to link EC2-Classic instances to a VPC using private IPv4 addresses (via an associated security group). Without it, EC2-Classic instances could only communicate with VPC instances over public IPv4.

---

## Exam Tip

If you see EC2-Classic or ClassicLink in an exam answer option, treat it as a **distractor**. The correct answer will involve VPC-based solutions.

---

## Quick Reference

```
EC2-Classic: pre-VPC shared network model — deprecated, cannot be used
ClassicLink: linked EC2-Classic to VPC via private IP — deprecated

Exam: both are distractors — always choose VPC-based answer
```
