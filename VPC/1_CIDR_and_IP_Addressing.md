# 1 — CIDR and IP Addressing

## What is CIDR?

Classless Inter-Domain Routing — a method to define IP address ranges.

Format: `<base IP>/<subnet mask>`
Example: `192.168.0.0/26` = 64 IP addresses

---

## Two Components

| Component | Description | Example |
|---|---|---|
| **Base IP** | Starting (or contained) IP of the range | `10.0.0.0`, `192.168.0.0` |
| **Subnet Mask** | How many bits can change | `/24`, `/16`, `/8` |

---

## Subnet Mask Quick Reference

| CIDR | IPs Available | Octets That Can Change | Range Example |
|---|---|---|---|
| /32 | 1 | None | single IP |
| /31 | 2 | last 1 bit | .0–.1 |
| /30 | 4 | — | .0–.3 |
| /28 | 16 | — | .0–.15 |
| /26 | 64 | — | .0–.63 |
| /24 | 256 | Last octet | .0–.255 |
| /16 | 65,536 | Last 2 octets | x.x.0.0–x.x.255.255 |
| /8 | 16,777,216 | Last 3 octets | x.0.0.0–x.255.255.255 |
| /0 | All IPs | All octets | 0.0.0.0–255.255.255.255 |

### Memory Shortcut

```
/32 → no octets change    → 1 IP
/24 → last 1 octet        → 256 IPs
/16 → last 2 octets       → 65,536 IPs
/8  → last 3 octets       → 16,777,216 IPs
/0  → all octets          → all IPs
```

---

## Common CIDR Examples

| CIDR | Meaning |
|---|---|
| `x.x.x.x/32` | Single IP |
| `0.0.0.0/0` | All IPv4 addresses |
| `192.168.0.0/24` | 256 IPs (.0–.255) |
| `192.168.0.0/16` | 65,536 IPs |

---

## Private IP Ranges (IANA)

| Range | CIDR | Use |
|---|---|---|
| `10.0.0.0` – `10.255.255.255` | `/8` | Large private networks |
| `172.16.0.0` – `172.31.255.255` | `/12` | AWS default VPC range |
| `192.168.0.0` – `192.168.255.255` | `/16` | Home networks |

All other IPv4 addresses are **public** (internet-routable).

---

## Quick Reference

```
CIDR = base IP + subnet mask (defines a range of IPs)

Key values:
  /32 → 1 IP       /24 → 256 IPs
  /16 → 65K IPs    /0  → all IPs

Private ranges:
  10.0.0.0/8        → large enterprise networks
  172.16.0.0/12     → AWS default VPC
  192.168.0.0/16    → home/small networks

Tool: use cidr.xyz or similar to convert CIDR ↔ IP range
```
