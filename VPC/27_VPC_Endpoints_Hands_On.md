# 27 — VPC Endpoints (Hands-On)

## Goal

Access Amazon S3 from a private EC2 instance (no internet) using a Gateway VPC Endpoint.

---

## Step 1: Attach IAM Role to Private EC2

**EC2 → Private instance → Security → Modify IAM role → Create new role**

| Setting | Value |
|---|---|
| Trusted entity | EC2 |
| Policy | `AmazonS3ReadOnlyAccess` |
| Role name | `DemoRoleEC2-S3ReadOnly` |

Attach to the private instance. Verify:
```bash
aws s3 ls    # should list buckets (while internet still attached)
```

---

## Step 2: Remove Internet Access from Private Subnet

**VPC → Route Tables → PrivateRouteTable → Routes → Edit routes**

Remove the `0.0.0.0/0 → NAT Gateway` route.

Verify no internet access:
```bash
curl google.com    # fails — no internet
aws s3 ls          # fails — no internet
```

---

## Step 3: Create S3 Gateway Endpoint

**VPC → Endpoints → Create endpoint**

| Setting | Value |
|---|---|
| Service category | AWS services |
| Service | `com.amazonaws.<region>.s3` — type: **Gateway** |
| VPC | DemoVPC |
| Route tables | **PrivateRouteTable** (check this box) |
| Policy | Full access |

Click **Create endpoint**.

AWS automatically adds a route to the private route table:

| Destination | Target |
|---|---|
| `pl-xxxxxxxx` (S3 prefix list) | `vpce-xxxxxxxx` (gateway endpoint) |

> This route cannot be manually deleted — it is managed by the endpoint.

---

## Step 4: Verify S3 Access Without Internet

```bash
# From private EC2 (via bastion SSH)

curl google.com           # still fails — no internet
aws s3 ls                 # may fail if CLI defaults to us-east-1

# Fix: specify the correct region explicitly
aws s3 ls --region eu-central-1    # succeeds — traffic routed via VPC endpoint
```

> **Gotcha:** AWS CLI defaults to `us-east-1`. If your resources are in another region, the CLI sends requests to the wrong regional endpoint. Always specify `--region` when testing VPC endpoints.

---

## Interface vs Gateway in the Console

When creating an endpoint, filtering by service name shows the type:
- `dynamodb` → **Gateway**
- `s3` → **Gateway** (preferred) or Interface
- Everything else → **Interface** (requires subnet + security group selection)

---

## Quick Reference

```
S3 Gateway Endpoint Hands-On:
  1. Attach IAM role with S3ReadOnly to private EC2
  2. Remove 0.0.0.0/0 → NAT GW route from PrivateRouteTable
  3. VPC → Endpoints → Create → S3 Gateway → select PrivateRouteTable
     AWS auto-adds route: S3 prefix list → gateway endpoint
  4. Test: aws s3 ls --region <your-region>

Gotcha: AWS CLI defaults to us-east-1 → add --region flag when testing
curl google.com still fails — only S3 is routed through the endpoint
```
