# 18 — VPC DNS Settings (Hands-On)

## Goal

Enable DNS hostnames on DemoVPC, then create a Route 53 private hosted zone and verify private DNS resolution from an EC2 instance.

---

## Step 1: Enable DNS Hostnames on the VPC

**VPC → Your VPCs → DemoVPC → Actions → Edit DNS hostnames → Enable → Save**

Effect: EC2 instances with public IPs now receive a public DNS hostname.

| Before | After |
|---|---|
| Public IP: ✓ | Public IP: ✓ |
| Public DNS: ✗ | Public DNS: `ec2-x-x-x-x.eu-central-1.compute.amazonaws.com` ✓ |

> DNS resolution (`enableDnsSupport`) was already enabled by default — verify via **Actions → Edit DNS resolution**.

---

## Step 2: Create a Private Hosted Zone

**Route 53 → Hosted zones → Create hosted zone**

| Setting | Value |
|---|---|
| Domain name | `demo.internal` |
| Type | **Private hosted zone** |
| Region | eu-central-1 |
| VPC | DemoVPC |

> You do NOT need to own a private hosted zone domain — it resolves only within the associated VPC.
> Cost: **$0.50/month** per hosted zone.

**Prerequisite:** The associated VPC must have both `enableDnsHostnames` and `enableDnsSupport` set to true.

---

## Step 3: Create a Test Record

**Hosted zone → Create record**

| Setting | Value |
|---|---|
| Record name | `google.demo.internal` |
| Type | CNAME |
| Value | `www.google.com` |
| Routing policy | Simple |

---

## Step 4: Verify DNS Resolution

SSH into the bastion host (public subnet) and test:

```bash
dig google.demo.internal
```

Expected output:
```
;; ANSWER SECTION:
google.demo.internal.   300   IN   CNAME   www.google.com.
www.google.com.         300   IN   A       <google IP>
```

The private hosted zone record resolves correctly within the VPC.

---

## Key Takeaways

- Private hosted zone domains (e.g., `demo.internal`) do **not** require domain ownership
- Resolution only works within the associated VPC — not from the public internet
- Enables meaningful internal DNS names for private applications (e.g., `db.demo.internal`, `api.demo.internal`)

---

## Quick Reference

```
Enable DNS Hostnames: VPC → Edit DNS hostnames → Enable
  Effect: public EC2 instances get public DNS names

Private Hosted Zone: Route 53 → Create hosted zone → Private → associate VPC
  Requires: enableDnsHostnames + enableDnsSupport = true on VPC
  Cost: $0.50/month

Verify: dig <record>.demo.internal → should return CNAME/A answer
```
