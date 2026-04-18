# 29 — CloudTrail: SysOps Exam Details

## Log File Integrity Validation

CloudTrail delivers log files to S3 every hour. You can enable **digest files** for tamper detection.

| Item | Detail |
|---|---|
| **Digest file** | References all log files from the last hour + SHA-256 hash of each |
| **Storage location** | Same S3 bucket, different folder from log files |
| **Purpose** | Prove a log file has not been modified or deleted after delivery |
| **Hash algorithm** | SHA-256 |

If log file hash = digest file hash → file is unmodified.

> Use digest files to satisfy **compliance requirements** that logs have not been tampered with.

### Additional S3 Protection for CloudTrail Logs

- Bucket policy
- Versioning
- MFA Delete
- Encryption
- S3 Object Lock (WORM)

---

## CloudTrail + EventBridge Integration

- CloudTrail feeds all API calls into EventBridge
- **Not real-time** — events delivered to EventBridge within **~15 minutes** of the API call
- Log files delivered to S3 within **~5 minutes**

```
API call → CloudTrail (≤15 min) → EventBridge rule → Lambda / SNS / SQS
                      └──────── (≤5 min) → S3 log file
```

> Use CloudTrail + EventBridge to react to **any** API call, including those not natively covered by EventBridge service events.

---

## Organization Trails

Set up a single CloudTrail trail at the **AWS Organizations** level to capture events across all accounts.

| Feature | Detail |
|---|---|
| **Scope** | Management account + all member accounts |
| **Destination** | Organization-wide S3 bucket |
| **Trail name** | Same in every account |
| **Member account permissions** | Can **view** the trail exists — cannot modify or delete it |
| **Compliance benefit** | Centralized, tamper-proof audit log across the entire org |

---

## SysOps Exam Q&A

**Q: How do you prove a CloudTrail log file has not been modified?**
A: Enable **log file integrity validation** — CloudTrail creates a **digest file** with SHA-256 hashes of each log file. Compare hashes to verify integrity.

**Q: Is CloudTrail + EventBridge real-time?**
A: **No** — events appear in EventBridge within ~15 minutes of the API call. Log files reach S3 within ~5 minutes.

**Q: What is an Organization Trail?**
A: A CloudTrail trail created at the AWS Organizations level that logs all API calls for the management account and all member accounts into a shared S3 bucket. Member accounts cannot modify or delete it.

**Q: How do you protect CloudTrail log files in S3 from tampering?**
A: Bucket policy, versioning, MFA Delete, encryption, and S3 Object Lock (WORM). Use digest files to prove integrity.

---

## Quick Reference

```
Log file integrity:
  Digest file = SHA-256 hash per log file, stored in same bucket/different folder
  Proves: log not modified after CloudTrail delivered it
  Algorithm: SHA-256

S3 protection: bucket policy + versioning + MFA Delete + encryption + Object Lock

CloudTrail + EventBridge: NOT real-time
  Events → EventBridge: ≤15 minutes
  Log files → S3: ≤5 minutes

Organization Trail:
  Covers management + all member accounts
  Same trail name across all accounts
  Members: view only — cannot modify or delete
  Destination: organization-wide S3 bucket
```
