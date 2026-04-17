# 12. AWS Global Accelerator — Hands-On

## Overview

A walkthrough of creating a Global Accelerator with EC2 instances in two regions (US-East-1 and EU-West-1), verifying Anycast routing sends users to the nearest region, and cleaning up.

---

## Architecture

```
Users (Europe)   ──► Anycast IP ──► EU-West-1 EC2 instance
Users (Americas) ──► Anycast IP ──► US-East-1 EC2 instance
                              ↑
              Same 2 static IPs, different destination based on proximity
```

---

## Step 1: Create the Accelerator

**Console:** AWS Global Accelerator (global service — no region selector) → **Create accelerator**

| Setting | Value |
|---------|-------|
| Name | `Demo` |
| Accelerator type | Standard |
| IP address type | IPv4 |

---

## Step 2: Configure Listener

| Setting | Value |
|---------|-------|
| Protocol | TCP |
| Port | 80 |
| Client affinity | None (or Source IP to pin users to same endpoint) |

---

## Step 3: Configure Endpoint Groups

Add one endpoint group per region:

| Endpoint Group | Region |
|----------------|--------|
| Group 1 | `us-east-1` |
| Group 2 | `eu-west-1` |

> Health check configuration: Protocol = HTTP, Path = `/`, Port = 80, Interval = 10s, Threshold = 2
> (Apply to both endpoint groups)

---

## Step 4: Launch EC2 Instances (One Per Region)

### US-East-1 Instance

| Setting | Value |
|---------|-------|
| Name | `DemoAccelerator` |
| AMI | Amazon Linux (latest) |
| Instance type | t3.micro |
| Key pair | None |
| Security group | Allow HTTP (port 80) from anywhere |
| User data | See below |

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "Hello World from $(hostname) in US-East-1" > /var/www/html/index.html
```

### EU-West-1 Instance

Same configuration, change user data to:

```bash
echo "Hello World from $(hostname) in EU-West-1" > /var/www/html/index.html
```

---

## Step 5: Add Endpoints to Each Group

Global Accelerator → Listener → Endpoint groups → Add endpoints

| Endpoint Group | Endpoint Type | Select |
|----------------|--------------|--------|
| us-east-1 | EC2 instance | Select the US-East-1 instance |
| eu-west-1 | EC2 instance | Select the EU-West-1 instance |

> **Weight** controls traffic distribution between multiple endpoints in the same group.

Wait for:
1. Accelerator status: **Deployed** (provisioning complete)
2. Health check status: **Healthy** (takes ~20 seconds with threshold = 2 and 10s interval)

---

## Step 6: Verify Anycast Routing

After provisioning, Global Accelerator shows **2 static Anycast IPs**.

```
IP 1: x.x.x.x  →  access from browser
IP 2: y.y.y.y  →  access from browser
```

**Expected behavior:**
- From Europe: both IPs resolve to the **EU-West-1** instance
- From US (via VPN or US-based connection): both IPs resolve to the **US-East-1** instance
- Keep refreshing — traffic stays on the same region (nearest edge wins)

---

## Step 7: Cleanup

> Always clean up to avoid charges — Global Accelerator is billed per accelerator hour.

1. Global Accelerator → select accelerator → **Disable**
2. Wait for status: Disabled
3. **Delete** the accelerator
4. EC2 → **us-east-1** → terminate `DemoAccelerator` instance
5. EC2 → **eu-west-1** → terminate the EU instance

---

## Best Practices

✓ **Configure health checks on all endpoint groups** — without them, unhealthy endpoints receive traffic  
✓ **Set threshold count to 2 with 10s interval** for faster health check convergence in demos  
✓ **Always disable before deleting** — Global Accelerator requires a disable step before deletion  
✓ **Terminate instances in both regions** — don't leave running instances after the demo  

---

## SysOps Exam Focus

**Q1: "You create a Global Accelerator with endpoints in us-east-1 and eu-west-1. A user in London accesses one of the Anycast IPs. Which endpoint receives the request?"**
- A) us-east-1, because it was configured first
- B) eu-west-1, because it is geographically closest to London
- C) Both, using round-robin load balancing
- D) Whichever endpoint responds first
- **Answer: B** — Anycast routes users to the nearest edge location, which then forwards to the closest endpoint group

**Q2: "After adding endpoints in Global Accelerator, the health status shows Unhealthy. What is the most common cause?"**
- A) The endpoint is in the wrong region
- B) The accelerator is not yet deployed (provisioning incomplete) or the health check path/port is misconfigured
- C) IAM permissions are missing
- D) The EC2 instance type is not supported
- **Answer: B** — Health checks fail if the accelerator is still provisioning or if the health check protocol/path/port doesn't match the application

**Q3: "What must you do before you can delete a Global Accelerator?"**
- A) Remove all endpoints first
- B) Delete the listener
- C) Disable the accelerator first, then delete it
- D) Nothing — you can delete directly
- **Answer: C** — Global Accelerator must be disabled before it can be deleted

---

## Quick Reference

```
Create accelerator:
  Name → Standard → IPv4
  Listener: TCP port 80
  Endpoint groups: one per region
  Endpoints: EC2 / ALB / NLB / Elastic IP

Health checks:
  Protocol: HTTP | Path: / | Port: 80
  Interval: 10s | Threshold: 2
  Wait ~20s for healthy status after deployment

Two static Anycast IPs provided → both route to nearest endpoint group

Cleanup order:
  1. Disable accelerator
  2. Delete accelerator
  3. Terminate EC2 instances in all regions
```

---

**File: 12_AWS_Global_Accelerator_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
