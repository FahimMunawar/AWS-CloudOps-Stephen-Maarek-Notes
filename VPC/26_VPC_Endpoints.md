# 26 — VPC Endpoints

## Overview

VPC endpoints allow EC2 instances to access AWS services **privately** over the AWS network — no internet gateway or NAT gateway required.

```
Without VPC endpoint:
  Private EC2 → NAT Gateway → Internet Gateway → AWS Service (public internet)

With VPC endpoint:
  Private EC2 → VPC Endpoint → AWS Service (private AWS network, never touches internet)
```

---

## Two Types of VPC Endpoints

| Feature | Interface Endpoint | Gateway Endpoint |
|---|---|---|
| Powered by | AWS PrivateLink | Route table entry |
| Provisions | ENI (private IP in your VPC) | Gateway target |
| Security Group | Required | Not used |
| Supported services | Most AWS services | **S3 and DynamoDB only** |
| Cost | Per hour + per GB processed | **Free** |
| Scales | Yes | Yes (automatic) |

---

## Interface Endpoint

- Provisions an **ENI** inside your VPC with a private IP
- Attach a security group to the ENI
- Entry point for traffic destined to the AWS service
- Covers almost all AWS services (SNS, CloudWatch, SQS, etc.)

---

## Gateway Endpoint

- Provisions a **gateway** that is added as a **route table target**
- No IP address, no security group
- **Only supports S3 and DynamoDB**
- **Free** — no per-hour or per-GB cost

```
Route table entry:
  Destination: pl-xxxxxxxx (S3 prefix list)  Target: vpce-xxxxxxxx (gateway)
```

---

## Gateway vs Interface for S3/DynamoDB (Exam Decision)

| Scenario | Preferred |
|---|---|
| Standard VPC access to S3 or DynamoDB | **Gateway Endpoint** (free, simpler) |
| Access from on-premises (via VPN or Direct Connect) | Interface Endpoint |
| Access from another VPC | Interface Endpoint |

> At the exam: Gateway Endpoint is almost always the right answer for S3 and DynamoDB within a VPC.

---

## Troubleshooting

If VPC endpoint connectivity fails, check:
1. **DNS resolution** settings on the VPC (`enableDnsSupport = true`)
2. **Route tables** — Gateway Endpoint requires a route table entry pointing to the endpoint

---

## SysOps Exam Q&A

**Q: An EC2 instance in a private subnet needs to access S3 without going through the internet. What is the most cost-effective solution?**
A: Gateway Endpoint for S3 — free, no NAT Gateway needed, just add a route table entry.

**Q: What are the only two services supported by Gateway Endpoints?**
A: Amazon S3 and DynamoDB.

**Q: What is required to use an Interface Endpoint?**
A: An ENI is provisioned in your VPC and a security group must be attached.

**Q: When would you prefer an Interface Endpoint over a Gateway Endpoint for S3?**
A: When accessing S3 from on-premises (via VPN/Direct Connect) or from another VPC — Gateway Endpoints are VPC-scoped only.

---

## Quick Reference

```
VPC Endpoints: private AWS network access — no IGW or NAT GW needed

Gateway Endpoint (preferred for S3/DynamoDB):
  Free, route table target, no SG, S3 + DynamoDB only

Interface Endpoint (everything else):
  ENI + private IP, security group required, per hour + per GB cost
  Also use for S3/DynamoDB when on-premises or cross-VPC access needed

Troubleshoot: check DNS resolution + route tables
```
