# 14 — KMS Key Rotation

## Three Rotation Methods

| Method | Key Type | Period | KMS Key ID Changes? |
|---|---|---|---|
| **Automatic (AWS-managed)** | AWS-managed keys | Every 1 year (fixed) | No |
| **Automatic (customer-managed)** | Customer-managed symmetric | 90–2,560 days (default: 1 year) | No |
| **On-demand** | Customer-managed symmetric | Any time (manual trigger) | No |
| **Manual** | Any (including asymmetric) | Any period | **Yes** |

---

## Automatic Key Rotation

- AWS-managed keys: always rotated every 1 year, cannot be changed
- Customer-managed symmetric keys: optional, configurable period (90–2,560 days)
- Previous **backing key** is retained — old encrypted data can still be decrypted
- **KMS Key ID stays the same** — only the backing key changes

```
Before rotation:  KMS Key ID: abc123  →  Backing Key v1
After rotation:   KMS Key ID: abc123  →  Backing Key v2 (v1 saved internally)
```

---

## On-Demand Key Rotation

- Customer-managed symmetric keys only
- Does **not** require automatic rotation to be enabled
- Does **not** affect existing automatic rotation schedule
- Has a **limit** on how many times it can be triggered
- KMS Key ID unchanged; new backing key created, old one saved

---

## Manual Key Rotation

Use when automatic rotation is not supported (e.g., **asymmetric keys**) or when a shorter rotation period is required (e.g., every 30 days).

**Process:**
1. Create a **new KMS key** → gets a new KMS Key ID
2. Update the **alias** to point to the new key (`UpdateAlias` API call)
3. Keep the old key **active** for decrypting previously encrypted data

```
Before:  Alias MyAppKey → Old Key (key-id-aaa)
After:   Alias MyAppKey → New Key (key-id-bbb)  ← UpdateAlias API call
                          Old Key (key-id-aaa)   ← kept active for decryption
```

Application continues to reference the alias — the key change is transparent.

---

## Exam Summary

| Requirement | Solution |
|---|---|
| Rotate every 1 year automatically | Automatic rotation (default) |
| Rotate every 90 or 180 days | **Manual rotation** with alias update |
| Rotate asymmetric key | **Manual rotation** only |
| Application must not notice key change | Use **alias** — update alias to new key |

---

## SysOps Exam Q&A

**Q: You need to rotate a KMS key every 90 days. What approach do you use?**
A: Manual key rotation — create a new key, update the alias to point to it. Automatic rotation minimum is 90 days but the exam typically frames short intervals as manual rotation.

**Q: After automatic key rotation, can you still decrypt data encrypted with the old key?**
A: Yes — the previous backing key is saved internally and used automatically for decryption.

**Q: What changes when automatic KMS key rotation occurs?**
A: Only the backing key changes. The KMS Key ID remains the same.

**Q: How do you rotate an asymmetric KMS key?**
A: Manually — create a new key and update the alias. Asymmetric keys are not eligible for automatic rotation.

---

## Quick Reference

```
Automatic rotation:
  AWS-managed: every 1 year (fixed)
  Customer-managed symmetric: 90–2,560 days (optional, default 1 year)
  KMS Key ID unchanged; backing key rotated; old backing key saved

On-demand rotation:
  Customer-managed symmetric only; no schedule required; KMS Key ID unchanged

Manual rotation:
  New key created → new KMS Key ID
  Use UpdateAlias to point alias to new key → app is transparent
  Required for: asymmetric keys, rotation < 90 days
  Keep old key active for decryption of existing data
```
