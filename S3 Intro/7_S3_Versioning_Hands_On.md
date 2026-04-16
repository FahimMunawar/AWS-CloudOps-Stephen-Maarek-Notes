# 7 — S3 Versioning — Hands-On

## Overview

Practical walkthrough of S3 versioning: uploading multiple versions, rolling back, using delete markers, and restoring deleted objects.

---

## Enabling Versioning

1. S3 → Bucket → **Properties** tab
2. **Bucket Versioning** → Edit → **Enable** → Save

After enabling, any new upload to an existing key creates a new version instead of overwriting.

---

## Viewing Versions

In the Objects tab, toggle **"Show versions"** to see:
- Each object's version IDs
- Delete markers
- `null` version IDs for objects uploaded before versioning was enabled

---

## Version ID Behaviour

| Scenario | Version ID |
|---|---|
| Object uploaded **before** versioning enabled | `null` |
| Object uploaded **after** versioning enabled | Unique alphanumeric ID (e.g., `abc123...`) |
| Same key uploaded again | New version ID created; old version preserved |

### Example: index.html uploaded twice
```
index.html  (uploaded before versioning)  → version: null
index.html  (uploaded after versioning)   → version: abc123xyz
```
Both versions coexist. The latest version is served by default.

---

## Rolling Back to a Previous Version

To undo a change and restore the previous version:

1. Enable **"Show versions"** toggle
2. Select the **latest version** (the one you want to remove)
3. Click **Delete**
4. Confirm with `permanently delete`

This is a **permanent delete of a specific version** — it cannot be undone.

Result: the previous version becomes the current (latest) version automatically.

---

## Delete Behaviour: With vs Without Version ID

### Without specifying a version (normal delete)
```
Delete coffee.jpg  (Show versions: OFF)
        │
        ▼
  Delete marker added (new "version" with no content)
  Previous version still exists underneath
  Object appears gone — returns 404 to clients
```

### With specifying a version ID (permanent delete)
```
Delete specific version ID
        │
        ▼
  That version is permanently removed
  Cannot be undone
```

---

## Restoring a Deleted Object (Delete Marker)

When an object has been "deleted" (delete marker added):

1. Enable **"Show versions"** toggle
2. Find the **delete marker** entry for the object (shows as a version with a delete marker icon)
3. Select the delete marker → **Delete** → `permanently delete`
4. The delete marker is removed → previous version is restored automatically

```
Delete marker removed
        │
        ▼
  Previous version becomes current again
  Object accessible again (200 OK)
```

---

## Two Types of Deletes — Summary

| Type | How | Reversible? | Effect |
|---|---|---|---|
| **Soft delete** | Delete without specifying version ID | Yes — delete the delete marker | Adds delete marker; object hidden but not gone |
| **Permanent delete** | Delete a specific version ID | **No** | Version permanently removed |

---

## SysOps Exam Q&A

**Q: You delete an S3 object in a versioning-enabled bucket without specifying a version ID. What happens?**
A: A **delete marker** is added. The object is not removed — all previous versions still exist. The object returns 404 to clients.

**Q: How do you restore an S3 object that was "deleted" with a delete marker?**
A: **Delete the delete marker** — this exposes the previous version and restores the object.

**Q: How do you permanently delete a specific version of an S3 object?**
A: Enable "Show versions", select the specific version ID, and delete it. Confirm with `permanently delete`. This is irreversible.

**Q: An S3 object shows version ID `null`. What does this mean?**
A: The object was uploaded **before versioning was enabled** on the bucket.

**Q: How do you roll back an S3 object to a previous version?**
A: Permanently delete the current (latest) version — the previous version automatically becomes the current version.

---

## Quick Reference

```
Show versions toggle   = reveals all version IDs and delete markers
null version ID        = uploaded before versioning was enabled
Soft delete            = delete marker added; reversible (delete the marker)
Permanent delete       = specific version ID deleted; irreversible
Rollback               = permanently delete the latest version → previous becomes current
Restore deleted object = delete the delete marker
404 on deleted object  = delete marker is present; previous versions still exist
```
