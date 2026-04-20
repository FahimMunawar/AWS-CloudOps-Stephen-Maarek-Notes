# 20 — Security Groups and NACLs (Hands-On)

## Goal

Demonstrate NACL rule priority, statelessness, and the difference from stateful security groups.

---

## Setup: HTTP Server on Bastion Host

```bash
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd
sudo su
echo "hello world" > /var/www/html/index.html
```

Add inbound HTTP (port 80) to bastion host security group → verify "hello world" loads in browser.

---

## Demo 1: NACL Rule Priority

**Default NACL → Inbound rules → Edit → Add rule:**

| Rule # | Type | Source | Action |
|---|---|---|---|
| 80 | HTTP (80) | `0.0.0.0/0` | **DENY** |
| 100 | All traffic | `0.0.0.0/0` | ALLOW |

**Result:** Browser times out — rule 80 (deny HTTP) fires before rule 100 (allow all).

**Change deny rule number to 140:**

| Rule # | Type | Source | Action |
|---|---|---|---|
| 100 | All traffic | `0.0.0.0/0` | ALLOW |
| 140 | HTTP (80) | `0.0.0.0/0` | DENY |

**Result:** Browser loads "hello world" — rule 100 (allow all) fires first; rule 140 never reached.

> **Key takeaway:** Lower rule number = higher priority. First match wins — an explicit DENY at rule 140 is irrelevant if rule 100 already allowed the traffic.

---

## Demo 2: NACL Statelessness

**Default NACL → Outbound rules → Edit → change allow to DENY**

**Result:** Browser times out — even though inbound was allowed, the outbound (return traffic) is blocked because NACLs are stateless.

> Security group outbound rules showed "allow all" — but that didn't help. The NACL outbound deny overrode everything.
> **Key takeaway:** If network connectivity fails but security group rules look correct, always check the NACL outbound rules for stateless blocking.

Revert: restore outbound ALLOW to fix.

---

## Demo 3: Security Group Statefulness

**Bastion host SG → Outbound rules → Remove all outbound rules**

**Result:** Browser still loads "hello world" — inbound-initiated traffic return is automatically allowed because security groups are stateful.

> Even with zero outbound rules, responses to accepted inbound connections go through. Statefulness means the SG tracks the connection and permits the return.

**Contrast:** If the EC2 instance tried to *initiate* an outbound connection (e.g., `curl google.com`), it would be denied — no outbound rule exists for self-initiated traffic.

Revert: restore HTTP outbound rule.

---

## Troubleshooting Tip

When diagnosing connectivity issues:
1. Check Security Group inbound rules
2. Check Security Group outbound rules (only matters for self-initiated connections)
3. Check **NACL inbound rules**
4. Check **NACL outbound rules** — stateless, must explicitly allow return traffic (including ephemeral ports)

---

## Quick Reference

```
NACL rule priority demo:
  Rule 80 DENY HTTP + Rule 100 ALLOW ALL → denied (80 wins)
  Rule 100 ALLOW ALL + Rule 140 DENY HTTP → allowed (100 wins)

NACL stateless demo:
  NACL outbound = DENY → browser times out even if inbound is allowed
  Security group rules are irrelevant if NACL blocks return traffic

SG stateful demo:
  Remove all SG outbound rules → inbound-initiated traffic still works
  Self-initiated outbound (curl) would be blocked — no outbound rule

Debugging order: SG inbound → SG outbound → NACL inbound → NACL outbound
```
