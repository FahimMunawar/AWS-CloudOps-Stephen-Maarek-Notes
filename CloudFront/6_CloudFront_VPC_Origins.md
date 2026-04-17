# 6. CloudFront VPC Origins

## Overview

**VPC Origins** is the modern, secure way to connect CloudFront to backend applications (ALB, NLB, or EC2 instances) running in **private subnets**, without exposing them to the internet.

---

## Two Methods: VPC Origins (New) vs Public Network (Old)

| | VPC Origins (New) | Public Network (Old) |
|-|-------------------|----------------------|
| **Backend subnet** | Private | Public (must be internet-facing) |
| **Security group management** | Not required | Must allow all CloudFront edge location IPs |
| **Security risk** | None — fully private | If SG is misconfigured, backend is publicly accessible |
| **Maintenance overhead** | Low | High — CloudFront IP list changes over time |
| **Supported backends** | Private ALB, NLB, EC2 | Public ALB, public EC2 |
| **Recommendation** | ✓ Preferred | Legacy |

---

## VPC Origins Architecture (New Method)

```
Users
  ↓
CloudFront Distribution (edge locations)
  ↓
VPC Origin (private connection)
  ↓
Private Subnet
  └── Private ALB / Private NLB / Private EC2
```

- The backend **never needs a public IP**
- CloudFront routes to the private resource through the VPC Origin
- Most secure architecture — attack surface is minimized

---

## Public Network Architecture (Old Method)

```
Users
  ↓
CloudFront edge locations (public IPs)
  ↓
Public ALB (security group: allow CloudFront IP list)
  ↓
Private EC2 instances (security group: allow ALB only)
```

**Problems with this approach:**
- ALB must be **public-facing** (exposed to internet)
- You must manually maintain a security group allowing all CloudFront edge location IPs
- CloudFront IP ranges can change — security group must be kept in sync
- A misconfigured security group leaves the ALB directly accessible from the internet

Reference: AWS publishes the CloudFront IP list at `https://ip-ranges.amazonaws.com/ip-ranges.json`

---

## Best Practices

✓ **Always use VPC Origins for new setups** — keeps backends private, no IP list management  
✓ **Migrate existing public ALB/EC2 origins to VPC Origins** when possible  
✓ **For legacy public setups**, use AWS-managed prefix lists in security groups to auto-track CloudFront IPs  
✓ **Keep EC2 instances in private subnets** even in the old method — only the ALB needs to be public  

---

## SysOps Exam Focus

**Q1: "You want CloudFront to serve traffic from an ALB in a private subnet. What feature should you use?"**
- A) Origin Access Control (OAC)
- B) VPC Origin
- C) Origin Shield
- D) Lambda@Edge
- **Answer: B** — VPC Origins connects CloudFront to private backends without requiring a public IP

**Q2: "Before VPC Origins existed, how was an EC2 instance secured as a CloudFront origin?"**
- A) The instance was kept private and accessed via OAC
- B) The instance was made public and its security group allowed only CloudFront edge location IP ranges
- C) The instance was accessed through a VPN
- D) The instance used IAM roles to authenticate with CloudFront
- **Answer: B** — The old method required a public EC2/ALB with a security group restricted to CloudFront's IP list

**Q3: "What is the main security risk of the old public-network CloudFront origin method?"**
- A) CloudFront cannot cache responses from EC2 origins
- B) A misconfigured security group can expose the ALB or EC2 directly to the public internet
- C) The CloudFront distribution becomes publicly accessible without authentication
- D) EC2 instances cannot be placed in private subnets
- **Answer: B** — If the SG is modified incorrectly, the backend becomes fully public rather than restricted to CloudFront only

**Q4: "Which backend resource types can be used with CloudFront VPC Origins?"**
- A) Only EC2 instances
- B) Only Application Load Balancers
- C) Private ALB, private NLB, and private EC2 instances
- D) Any resource with a private IP address, including RDS
- **Answer: C** — VPC Origins supports private ALB, NLB, and EC2 instances in private subnets

---

## Quick Reference

```
VPC Origins (preferred):
  Backend stays private → no public IP needed
  CloudFront → VPC Origin → private ALB/NLB/EC2
  No security group maintenance required

Public network (legacy):
  Backend must be public → ALB/EC2 with public IP
  Security group must allow all CloudFront edge IPs
  Risk: misconfiguration exposes backend to internet

Use VPC Origins for all new setups.
```

---

**File: 6_CloudFront_VPC_Origins.md**
**Status: SysOps-focused, exam-ready, concise format**
