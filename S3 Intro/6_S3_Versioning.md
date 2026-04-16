# 6 — S3 Versioning

## Overview

S3 Versioning allows you to keep multiple versions of an object in the same bucket. It is a **bucket-level setting** and is considered best practice for any bucket where data safety matters.

---

## How Versioning Works

```
Upload file.txt (v1)    →  Version 1 created
Re-upload file.txt (v2) →  Version 2 created (v1 preserved)
Re-upload file.txt (v3) →  Version 3 created (v1, v2 preserved)
```

- Each upload to the **same key** creates a new version instead of overwriting
- All previous versions are retained
- You can retrieve, restore, or delete any specific version

---

## Why Enable Versioning (Best Practices)

| Benefit | Detail |
|---|---|
| **Protection against unintended deletes** | Deleting a versioned object adds a **delete marker** — the object is not actually removed |
| **Easy rollback** | Restore any object to a previous version at any time |
| **Audit history** | Full history of every change to every object |

---

## Delete Behaviour with Versioning

```
Delete file.txt (versioning ON)
        │
        ▼
  Delete marker added  ← object appears deleted but all versions still exist
        │
  Restore: delete the delete marker → object reappears
```

- A **delete marker** is a special placeholder version
- The object is not permanently deleted — all versions remain
- To permanently delete: you must explicitly delete a **specific version ID**

---

## Important Notes

| Scenario | Behaviour |
|---|---|
| File uploaded **before** versioning was enabled | Gets version ID = **`null`** |
| **Suspending** versioning | Does **not** delete existing versions — safe operation |
| **Disabling** vs **Suspending** | You can only suspend versioning, not fully disable it once enabled |

---

## Enabling Versioning

1. S3 → Bucket → **Properties** tab
2. **Bucket Versioning** → Edit → Enable → Save

---

## Best Practices

- Enable versioning on all buckets that contain important or production data
- Combine versioning with **S3 Lifecycle policies** to automatically expire old versions and control storage costs
- Use versioning alongside **MFA Delete** for extra protection against accidental permanent deletion
- Pre-versioning objects get version `null` — handle this in lifecycle rules if needed

---

## SysOps Exam Q&A

**Q: What happens when you delete an object in a versioning-enabled S3 bucket?**
A: A **delete marker** is added. The object is not permanently deleted — all versions are preserved and recoverable.

**Q: How do you permanently delete a versioned S3 object?**
A: Delete the **specific version ID** explicitly. Deleting without specifying a version ID only adds a delete marker.

**Q: What version ID is assigned to objects that existed before versioning was enabled?**
A: Version ID = **`null`**.

**Q: You suspend versioning on a bucket. What happens to existing versions?**
A: Existing versions are **preserved** — suspension does not delete previous versions. It only stops new versions from being created.

**Q: Why is enabling versioning considered best practice?**
A: It protects against accidental deletes (via delete markers) and allows rollback to any previous version of an object.

---

## Quick Reference

```
Enable versioning   = bucket-level setting (Properties tab)
Each overwrite      = creates a new version, preserves old
Delete (versioned)  = adds delete marker, does NOT remove versions
Permanent delete    = must specify a version ID explicitly
Pre-versioning IDs  = null (objects uploaded before versioning was on)
Suspend versioning  = safe; existing versions kept, new versions stop
```
