# 23 — VPC Reachability Analyzer (Hands-On)

## Cost Warning

**$0.10 per analysis run.** Use sparingly — the per-analysis cost is intentional to discourage continuous polling.

---

## Creating an Analysis Path

**VPC → Reachability Analyzer → Create and analyze path**

| Field | Value |
|---|---|
| Source type | Instance |
| Source | BastionHost |
| Destination type | Instance |
| Destination | Private Instance |
| Destination port | 443 (HTTPS) |
| Protocol | TCP |

Click **Create and analyze path** — no network traffic is sent; AWS analyzes configuration only.

---

## Demo 1: Security Group Missing HTTPS Rule

**Result:** Not reachable

**Explanation shown:** "None of the inbound rules in the following security group apply" — HTTPS (443) not in private instance security group (only port 22/SSH existed).

**Fix:** Edit private instance SG → add inbound HTTPS (443) from anywhere → re-run analysis.

**Result after fix:** Reachable ✓

Path shown in analyzer:
```
BastionHost EC2 → ENI → Security Group → NACL → NACL → Security Group → ENI → Private Instance
```

---

## Demo 2: NACL Blocking HTTPS

**Setup:** Default NACL → Edit inbound rules → Add rule 80: DENY HTTPS (`0.0.0.0/0`)

**Re-run analysis → Result:** Not reachable

**Explanation shown:** "The network ACL does not allow inbound traffic from the subnet to VPC" — NACL rule 80 denies HTTPS before rule 100 allows all traffic.

**Fix:** Remove the deny rule from NACL inbound rules → re-run.

---

## Key Observations

- Analyzer pinpoints the **exact component** causing the block (SG vs NACL vs route table)
- Shows the full hop-by-hop path when reachable
- Becomes increasingly valuable as topology grows (NAT GW, TGW, peering, egress-only IGW)

---

## Quick Reference

```
VPC Reachability Analyzer Hands-On:
  Cost: $0.10 per analysis — use deliberately
  Source/destination: instances, TGWs, IGWs, VPC endpoints, etc.
  Optionally specify destination port + protocol

Result: Reachable or Not Reachable + exact blocking component
  SG missing rule → "no inbound rules apply"
  NACL deny rule → "network ACL does not allow inbound traffic"

Best used when: complex topology makes manual config inspection slow
```
