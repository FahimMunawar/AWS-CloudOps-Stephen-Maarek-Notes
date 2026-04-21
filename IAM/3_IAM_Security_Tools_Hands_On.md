# 3 — IAM Security Tools (Hands-On)

## Credentials Report

**IAM → Credential report → Download credential report** → downloads as CSV

### Columns Included

| Column | Description |
|---|---|
| User | Username (root account + all IAM users) |
| Password enabled | Whether password login is active |
| Password last used | Last console login date |
| Password last changed | Last password change date |
| Password next rotation | Date if rotation policy is enabled |
| MFA active | Whether MFA is enabled for the user |
| Access key 1/2 active | Whether access keys exist |
| Access key last rotated | Age of the access key |
| Access key last used | Last programmatic API activity |

### Use Case

Identify security risks at a glance:
- Users with old/unused passwords
- Users without MFA enabled
- Access keys that haven't been rotated

---

## IAM Access Advisor

**IAM → Users → [select user] → Access Advisor tab**

Shows every AWS service the user has permission to access, with the **last accessed date** and **which policy granted access**.

### Example Output

| Service | Last Accessed | Granted By |
|---|---|---|
| Amazon EC2 | Recently | AdministratorAccess |
| AWS Organizations | Recently | AdministratorAccess |
| Alexa for Business | Never | AdministratorAccess |
| AWS App2Container | Never | AdministratorAccess |

Services never accessed → candidates for permission removal.

### Use Case

Enforce least privilege — if a user has 37 pages of service permissions but only accesses 5 services, reduce the policy to cover only what is actually used.

---

## Quick Reference

```
Credentials Report: IAM → Credential report → Download CSV
  Account-wide; shows password age, MFA status, access key age/last used
  Use: find stale credentials, users without MFA

Access Advisor: IAM → User → Access Advisor tab
  Per-user; shows service permissions + last accessed date + granting policy
  Use: identify unused permissions → trim to least privilege
```
