# 10 — AWS Service Catalog

## Problem It Solves

New AWS users face too many options and may create non-compliant resources. Service Catalog gives them a self-service portal with a curated, pre-approved list of products they can launch safely.

---

## Core Concepts

| Concept | Detail |
|---|---|
| **Product** | A CloudFormation template (encapsulated and versioned) |
| **Portfolio** | A collection of products (CloudFormation templates) |
| **IAM Permissions** | Control who can see and launch which portfolios |
| **Provisioned Product** | A launched, ready-to-use stack from a product |

---

## Workflow

```
Admin:
  Create Products (CloudFormation templates)
    → Group into Portfolios
      → Assign IAM Permissions to portfolios

User:
  Sees product list (filtered by IAM permissions)
    → Launches a product
      → Provisioned stack: properly configured, compliant
```

> Users may have **no direct AWS access** — they interact only with Service Catalog and still get full stacks launched on their behalf.

---

## Portfolio Sharing (Cross-Account)

| Method | Behavior |
|---|---|
| **Share reference** | Recipient imports portfolio — stays **in sync** with source; new products added in account A appear automatically in account B |
| **Deploy a copy** | Portfolio is copied to recipient — updates in account A must be **manually copied** to account B |

- Admins in recipient accounts can launch products from the imported/copied portfolio
- Products from imported portfolios can be added to local portfolios

---

## TagOptions

- Predefined key-value tag pairs managed in Service Catalog
- Associate a TagOption with a **portfolio** or **product**
- Any stack launched from that portfolio/product **inherits the tags automatically**

```
TagOption: Key=environment, Value=prod
  → Associate with Portfolio A
    → Every EC2, RDS, etc. launched from Portfolio A
        gets tag: environment=prod
```

**Use cases:**
- Enforce consistent resource tagging
- Only allow pre-approved tags (controlled via Service Catalog)
- TagOptions can be shared across accounts or within an organization

---

## SysOps Exam Q&A

**Q: How do you allow users to launch AWS resources without giving them direct access to CloudFormation or the AWS console?**
A: Use **AWS Service Catalog** — users access only the Service Catalog portal, select from approved products (CloudFormation templates), and launch compliant stacks.

**Q: What is the difference between sharing a portfolio reference vs. deploying a copy?**
A: A **shared reference** stays in sync — new products in the source appear automatically in the recipient. A **copy** is independent — updates must be manually copied.

**Q: How do you ensure all resources launched from a portfolio get a specific tag?**
A: Use **TagOptions** — associate a key-value tag with the portfolio; all provisioned products inherit it automatically.

---

## Quick Reference

```
Service Catalog:
  Product = CloudFormation template
  Portfolio = collection of products
  Access = IAM permissions on portfolio
  Provisioned = launched, compliant, ready-to-use stack

Users launch stacks without direct AWS/CloudFormation access

Portfolio sharing:
  Reference (in sync)  → new products appear automatically in recipient
  Copy (independent)   → updates must be manually pushed

TagOptions:
  Predefined key-value tags
  Attach to portfolio or product → auto-applied to all launched resources
  Shareable across accounts/org
```
