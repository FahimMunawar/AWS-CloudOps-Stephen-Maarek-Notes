# 18 — AWS Certificate Manager (Hands-On)

## Goal

Request a public TLS certificate via ACM, attach it to an ALB via Elastic Beanstalk, and verify HTTPS access.

---

## Step 1: Request a Public Certificate

**ACM → Request certificate → Public certificate**

| Setting | Value |
|---|---|
| FQDN | `acmdemo.yourdomain.com` |
| Validation method | **DNS validation** (preferred) |
| Key algorithm | Default (RSA 2048) |

Status: **Pending validation**

---

## Step 2: Create DNS Validation Record

**ACM → Certificate → Domains → Create records in Route 53**

ACM automatically creates the required CNAME record in the Route 53 hosted zone. Wait a few minutes for status to change to **Issued**.

> If the domain is not managed by Route 53, manually add the CNAME record to your DNS provider.

---

## Step 3: Launch Elastic Beanstalk with HTTPS Listener

**Elastic Beanstalk → Create application → Web server environment**

| Setting | Value |
|---|---|
| Platform | Node.js (managed) |
| Configuration | High availability (custom) |
| Load balancer type | Application Load Balancer (dedicated) |

**Add HTTPS listener:**

| Port | Protocol | SSL Certificate |
|---|---|---|
| 443 | HTTPS | Select ACM certificate from Step 1 |
| TLS policy | Default (e.g., ELBSecurityPolicy-2016-08) | |

Submit → wait for environment to launch.

---

## Step 4: Create CNAME in Route 53

**Route 53 → Hosted zone → Create record**

| Setting | Value |
|---|---|
| Record name | `acmdemo` |
| Type | CNAME |
| Value | Elastic Beanstalk environment URL (without `https://`) |

Wait for DNS propagation, then access `https://acmdemo.yourdomain.com`.

---

## Verification

- Browser shows padlock → connection is secure
- Certificate details show issued by Amazon (ACM)
- **EC2 → Load Balancers → Listeners and rules → HTTPS:443** → default SSL/TLS certificate = ACM certificate

---

## Cleanup

**Elastic Beanstalk → Application → Actions → Delete Application**

Type the full application name to confirm. This removes the ALB, ASG, and EC2 instances.

---

## Quick Reference

```
ACM Hands-On Flow:
  1. ACM → Request public cert → FQDN → DNS validation
  2. Create CNAME validation record in Route 53 (one click) → wait for Issued
  3. Beanstalk → high availability → ALB → add listener port 443 HTTPS → select ACM cert
  4. Route 53 → CNAME acmdemo → Beanstalk URL
  5. Access https://acmdemo.yourdomain.com → padlock visible

Verify in console: EC2 → Load Balancers → Listeners → HTTPS:443 → default cert = ACM
Cleanup: Beanstalk → Delete Application
```
