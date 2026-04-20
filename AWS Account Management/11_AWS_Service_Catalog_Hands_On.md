# 11 — AWS Service Catalog: Hands-On

## Admin Workflow

### 1. Create a Product

**Product List → Create Product → CloudFormation** (also supports Terraform)

| Field | Detail |
|---|---|
| Product name | e.g., `LAMP Stack` |
| Owner | Publisher name |
| Version | Upload a `.json`/`.yaml` CloudFormation template file |
| Version name | e.g., `First Version` |
| Support details | Contact info for users who need help |

### 2. Create a Portfolio

**Portfolios → Create portfolio**
- Name (e.g., `WebDevPortfolio`), creator name
- Portfolios are **local** (own account) or **imported** (from another account)

### 3. Add Product to Portfolio

Inside the portfolio → **Add products** → select existing product

### 4. Grant Access

Portfolio → **Access → Grant access**

| Option | Use Case |
|---|---|
| **IAM Principal** | Users/groups/roles in your own account |
| **Principal Name** | For sharing with other accounts |

Add individual IAM users, groups, or roles.

---

## User Workflow

1. Log in as the IAM user who was granted access
2. Go to **Service Catalog → Products**
3. Visible products are filtered by IAM permissions
4. Select product → **Launch product**
5. Fill in CloudFormation parameters (instance type, DB name, password, etc.)
6. Product provisions → CloudFormation stack is created behind the scenes
7. View **Resources**, **Events**, and **Outputs** (e.g., website URL) from the provisioned product page

---

## Behind the Scenes

```
User launches product in Service Catalog
  → Service Catalog calls CloudFormation
    → Stack created (named SC-<accountid>-<product>)
      → Resources provisioned + outputs available
```

> Users never touch CloudFormation directly — they only interact with Service Catalog.

---

## Cleanup

Terminate the provisioned product in Service Catalog → CloudFormation stack is deleted automatically.

---

## Optional: Custom Branding

**Preferences → Branding** — upload a company logo, set accent color, set brand name.
Available only in supported regions; requires submitting a support ticket.

---

## Quick Reference

```
Admin path:
  Create product (CloudFormation template)
    → Add to portfolio
      → Grant IAM access to portfolio

User path:
  Service Catalog → Products (filtered by permissions)
    → Launch → fill CloudFormation params
      → Provisioned product (stack created, outputs visible)

Supported product types: CloudFormation, Terraform
Cleanup: terminate provisioned product → deletes CloudFormation stack
```
