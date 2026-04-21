# 22 — Secrets Manager vs SSM Parameter Store

## Comparison

| Feature | Secrets Manager | SSM Parameter Store |
|---|---|---|
| **Cost** | Higher ($0.40/secret/month) | Lower (free tier + Standard/Advanced tiers) |
| **KMS encryption** | **Mandatory** | Optional |
| **Secret rotation** | Native (automatic via Lambda) | Manual (DIY with EventBridge + Lambda) |
| **RDS/Redshift/DocumentDB integration** | Built-in Lambda functions provided by AWS | Must write own Lambda |
| **API** | Secrets-focused | General-purpose parameters |
| **CloudFormation integration** | Yes | Yes |
| **Pull Secrets Manager secret via SSM API** | Yes (cross-service) | — |

---

## Rotation: Side-by-Side

### Secrets Manager (Native)

```
Secrets Manager
  → every 30 days (scheduled automatically)
    → invokes AWS-provided Lambda (e.g., for RDS)
      → Lambda changes RDS password
        → Secrets Manager stores new value
```

- For RDS, Redshift, DocumentDB: Lambda provided and deployed by AWS
- For other secrets: bring your own Lambda (AWS provides docs/templates)

### SSM Parameter Store (DIY)

```
EventBridge rule (every 30 days)
  → invokes custom Lambda function (you write it)
    → Lambda changes RDS password
    → Lambda updates SSM Parameter Store value
```

- No native rotation feature
- All rotation logic must be implemented manually

---

## When to Use Which

| Scenario | Use |
|---|---|
| Secrets + automatic rotation (especially RDS) | **Secrets Manager** |
| Simple config values / non-sensitive parameters | SSM Parameter Store |
| Cost-sensitive workload | SSM Parameter Store |
| Mandatory KMS encryption required | Secrets Manager |
| General application config (mixed secure + plain) | SSM Parameter Store |

---

## SysOps Exam Q&A

**Q: You need to automatically rotate an RDS database password every 30 days with minimal custom code. What do you use?**
A: Secrets Manager — it provides AWS-managed Lambda functions for RDS rotation out of the box.

**Q: Is KMS encryption required in Secrets Manager?**
A: Yes — all secrets in Secrets Manager are KMS-encrypted (mandatory).

**Q: How do you implement secret rotation using SSM Parameter Store?**
A: Create an EventBridge rule on a schedule → invoke a custom Lambda function that updates both the target service and the SSM Parameter Store value.

**Q: Can you retrieve a Secrets Manager secret using the SSM Parameter Store API?**
A: Yes — Secrets Manager secrets can be pulled via the SSM Parameter Store API.

---

## Quick Reference

```
Secrets Manager:
  Expensive, mandatory KMS, native rotation (Lambda provided for RDS/Redshift/DocumentDB)
  Best for: secrets + rotation + DB integration

SSM Parameter Store:
  Cheaper, optional KMS, no native rotation
  DIY rotation: EventBridge (schedule) → Lambda → update service + Parameter Store
  Best for: general config, non-DB secrets, cost-sensitive

Both: CloudFormation integration, KMS support
```
