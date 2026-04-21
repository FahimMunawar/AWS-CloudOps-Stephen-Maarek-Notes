# 2 — IAM Security Tools

## Two Security Tools

| Tool | Scope | Purpose |
|---|---|---|
| **IAM Credentials Report** | Account-level | Lists all IAM users and status of their credentials |
| **IAM Access Advisor** | User-level | Shows service permissions granted and when last used |

---

## IAM Credentials Report

- Generated from **IAM → Credential Report → Download**
- CSV file covering all users in the account
- Includes: password last used, access key age, MFA enabled, last activity, etc.
- Use case: audit credential hygiene, identify stale/unused credentials

---

## IAM Access Advisor

- Found on individual **IAM user → Access Advisor tab**
- Shows every AWS service the user has permission to access
- Shows the **last accessed date** for each service
- Use case: identify unused permissions → remove them → enforce **least privilege**

---

## SysOps Exam Q&A

**Q: You want to find all IAM users who have not used their access keys in 90 days. What tool do you use?**
A: IAM Credentials Report — it provides account-wide credential activity including access key last used dates.

**Q: You want to reduce a specific user's permissions to only what they actually use. What tool helps?**
A: IAM Access Advisor — it shows which services the user has permissions for and the last time each was accessed, allowing you to remove unused permissions.

---

## Quick Reference

```
Credentials Report (account-level):
  IAM → Credential Report → Download CSV
  All users + credential status (password age, access keys, MFA, last activity)
  Use: audit stale/unused credentials

Access Advisor (user-level):
  IAM → User → Access Advisor tab
  Service permissions + last accessed date per service
  Use: enforce least privilege by removing unused permissions
```
