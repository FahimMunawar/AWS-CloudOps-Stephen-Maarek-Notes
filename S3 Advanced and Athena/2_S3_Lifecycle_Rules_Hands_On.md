# 2 — S3 Lifecycle Rules — Hands-On

## Overview

Console walkthrough for creating lifecycle rules with transition and expiration actions. The console provides a visual timeline of what will happen to current and non-current versions.

---

## Creating a Lifecycle Rule (Console)

1. S3 → Bucket → **Management** tab → **Lifecycle rules** → **Create lifecycle rule**
2. Name the rule and choose scope (entire bucket, prefix, or tags)
3. Add **Transition actions** for current and/or non-current versions
4. Add **Expiration actions** for current and/or non-current versions
5. Review the **visual timeline** shown at the bottom
6. Click **Create rule** — an IAM role is created automatically to execute the rule in the background

---

## Example Rule Configuration

| Action | Target | Days |
|---|---|---|
| Transition current version | → Glacier Flexible Retrieval | After 90 days |
| Expire current versions | Permanently delete | After 700 days |
| Permanently delete non-current versions | Permanently delete | After 700 days |

---

## Visual Timeline

The console shows a **timeline diagram** after you configure actions, illustrating:
- What happens to the **current version** over time
- What happens to **non-current versions** over time
- Exact day numbers for each transition and expiration

This is useful for validating your rule before saving.

---

## Key Notes

- The lifecycle rule runs **in the background** via an automatically created IAM role
- You can combine multiple transitions and expirations in a single rule
- **Current version** = the latest version of the object
- **Non-current version** = older versions (only relevant when versioning is enabled)

---

## Quick Reference

```
Console path: S3 → Bucket → Management → Lifecycle rules → Create
Scope:        Entire bucket, prefix, or object tags
Actions:      Transition (change storage class) + Expiration (delete)
Timeline:     Visual diagram shown before saving — verify before creating
Background:   Rule executes via auto-created IAM role
```
