# 5 — AWS Organizations: Hands-On

## Setup

1. Management account → **AWS Organizations** → **Create an organization**
2. Organization is created instantly with **Root OU** containing the management account

---

## Adding a Member Account

Two options under **Add an account**:

| Option | Required Info |
|---|---|
| **Create a new account** | Account name, owner email, IAM role name (for org management) |
| **Invite existing account** | Email address or Account ID of the target account |

### Invitation Flow
1. Management account sends invitation
2. Child account: **Organizations → Invitations** → Accept
3. Accepted account appears under Root in the management account's **AWS Accounts** view
4. Invitations expire after **2 weeks** if not accepted
5. Member accounts can **leave** the organization at any time

---

## Creating OUs

**AWS Accounts → Root → Actions → Create new OU**

- Name the OU (e.g., Dev, Test, Prod)
- Create nested OUs within OUs (e.g., Prod → Finance, Prod → HR)
- No limit on nesting depth

### Example Hierarchy Built in Demo

```
Root
  ├── Management Account
  ├── Dev OU
  ├── Test OU
  └── Prod OU
        ├── Finance OU
        │     └── course-child (member account)
        └── HR OU
```

**Moving an account into an OU**: select account → **Actions → Move** → choose target OU

> Best practice: leave the management account directly under Root OU.

---

## Policy Types

Go to **Policies** to enable policy types (all disabled by default):

| Policy Type | Purpose |
|---|---|
| **Service Control Policies (SCPs)** | Restrict which AWS services accounts can use |
| **Backup Policies** | Deploy organization-wide backup plans |
| **Tag Policies** | Standardize tag usage across all accounts |

> For the exam: SCPs are the most important policy type.

---

## Service Control Policies (SCPs)

### Default Policy
When SCPs are enabled, a **FullAWSAccess** policy is auto-attached to Root (and all children inherit it).

### Creating a Custom SCP

**Policies → Service Control Policies → Create policy**

Example — Deny S3 access:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyS3",
      "Effect": "Deny",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
```

### Attaching an SCP

Select the target OU → **Policies → Attach** → choose the SCP

The SCP is then **inherited** by all child OUs and accounts below.

---

## SCP Inheritance Demo

| Account | Policies Applied | Result |
|---|---|---|
| Root OU | FullAWSAccess (attached) | Allow all |
| Prod OU | FullAWSAccess (inherited from Root + attached directly) | Allow all |
| Finance OU | FullAWSAccess (inherited ×2) + **DenyAccessS3** (attached) | Allow all except S3 |
| course-child | FullAWSAccess (inherited ×3) + DenyAccessS3 (inherited from Finance) | **Cannot use S3** |

> Even the root user of the child account cannot access S3 — SCPs override everything including root user permissions within the member account.

---

## Quick Reference

```
Create org → Root OU created → management account sits in Root
Add accounts: create new OR invite existing (expires in 2 weeks)

OUs: nested freely → Actions → Create new OU
Move account: select account → Actions → Move

Policy types: enable per type (SCPs most important for exam)
SCPs:
  Default: FullAWSAccess on Root (inherited by all)
  Custom: create policy → attach to OU or account
  Inheritance: policy on parent OU → inherited by all children
  Deny overrides all — even root user of member account is blocked
```
