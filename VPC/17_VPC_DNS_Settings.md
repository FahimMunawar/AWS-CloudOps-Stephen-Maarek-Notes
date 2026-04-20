# 17 — VPC DNS Settings

## Two DNS Attributes

| Attribute | Default (Default VPC) | Default (New VPC) | Purpose |
|---|---|---|---|
| `enableDnsSupport` | true | true | Enables Route 53 Resolver for DNS queries |
| `enableDnsHostnames` | true | **false** | Assigns public DNS hostname to EC2 instances with public IPs |

---

## enableDnsSupport

When **true**, EC2 instances can query the AWS-managed DNS resolver at:
- `169.254.169.253`
- VPC base CIDR + 2 (e.g., `10.0.0.2` for a `10.0.0.0/16` VPC)

```
EC2 Instance
  → Route 53 Resolver (169.254.169.253)
    → resolves google.com → public IP
      → internet access via IGW
```

When **false** → must configure and manage a custom DNS server.

---

## enableDnsHostnames

Requires `enableDnsSupport = true` to have any effect.

When **true**, EC2 instances with a public IP receive a **public DNS hostname**:

| Setting | EC2 DNS |
|---|---|
| DNS Hostnames disabled | Private DNS only (even in public subnet) |
| DNS Hostnames enabled | Private DNS **+** public DNS (e.g., `ec2-1-2-3-4.compute-1.amazonaws.com`) |

---

## Why Enable Both? — Private Hosted Zone Use Case

When both settings are true, EC2 instances can resolve **custom private DNS names** via Route 53 private hosted zones:

```
Private Hosted Zone: mycompany.private
  Record: web.mycompany.private → A → 10.0.0.10

EC2 Instance (private subnet)
  → queries web.mycompany.private
    → Route 53 Resolver
      → looks up private hosted zone
        → returns 10.0.0.10
          → EC2 reaches the target instance
```

This allows internal services to use human-readable DNS names instead of raw private IPs.

---

## SysOps Exam Q&A

**Q: An EC2 instance in a public subnet has a public IP but no public DNS name. What setting needs to be enabled?**
A: `enableDnsHostnames` (also requires `enableDnsSupport = true`).

**Q: What IP does the Route 53 Resolver listen on within a VPC?**
A: `169.254.169.253` or the VPC base CIDR + 2 (e.g., `10.0.0.2`).

**Q: What is the default value of `enableDnsHostnames` for a newly created VPC?**
A: **false** — must be manually enabled (default VPC has it true).

**Q: What is required to use a Route 53 private hosted zone with EC2 instances?**
A: Both `enableDnsSupport` and `enableDnsHostnames` must be true on the VPC.

---

## Quick Reference

```
enableDnsSupport (default: true):
  Allows EC2 to use Route 53 Resolver at 169.254.169.253 or VPC_CIDR+2
  Without it: must run custom DNS server

enableDnsHostnames (default: false for new VPCs):
  Gives EC2 instances a public DNS hostname (requires enableDnsSupport=true)
  Without it: EC2 has private DNS only, even with public IP

Both true → Route 53 private hosted zones work with VPC EC2 instances
```
