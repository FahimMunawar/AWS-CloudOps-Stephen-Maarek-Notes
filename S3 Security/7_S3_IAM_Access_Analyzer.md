# 7 — IAM Access Analyzer for S3

## Overview

IAM Access Analyzer for S3 monitors your S3 buckets and surfaces unintended public or cross-account access — powered by AWS IAM Access Analyzer.

---

## What It Analyzes

| Policy Type | Analyzed? |
|---|---|
| S3 Bucket Policies | Yes |
| S3 ACLs | Yes |
| S3 Access Point Policies | Yes |

---

## What It Detects

- Buckets **publicly accessible** (accessible by anyone on the internet)
- Buckets **shared with other AWS accounts** (cross-account access)
- Any access that was not intended

---

## How to Use It

1. IAM Access Analyzer surfaces findings in the console
2. Review each finding and decide: **intended** or **unintended**
3. If unintended → take action (update bucket policy, remove ACL, etc.)

---

## SysOps Exam Q&A

**Q: What is IAM Access Analyzer for S3 used for?**
A: To automatically analyze S3 bucket policies, ACLs, and access point policies to detect buckets that are **publicly accessible** or **shared with unintended AWS accounts**.

**Q: What policies does IAM Access Analyzer for S3 evaluate?**
A: **Bucket policies**, **S3 ACLs**, and **S3 Access Point policies**.

---

## Quick Reference

```
IAM Access Analyzer for S3:
  - Monitors: bucket policies, ACLs, access point policies
  - Detects:  public access, unintended cross-account sharing
  - Action:   review findings → mark as intended or remediate
  - Powered by: IAM Access Analyzer (finds resources shared with external entities)
```
