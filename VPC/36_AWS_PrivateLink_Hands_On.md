# 36 — AWS PrivateLink (Hands-On)

## Two-Part Setup

### Part 1: Service Provider — Create Endpoint Service

**VPC → Endpoint Services → Create endpoint service**

| Setting | Value |
|---|---|
| Load balancer type | Network |
| Load balancer | Select existing NLB (must be created first) |
| Acceptance required | Optional — require manual approval of consumer requests |
| Supported IP | IPv4 / IPv6 |

After creation, note the **service name** (e.g., `com.amazonaws.vpce.eu-central-1.vpce-svc-xxxxxxxx`) — consumers need this to connect.

---

### Part 2: Consumer — Create Endpoint

**VPC → Endpoints → Create endpoint**

| Setting | Value |
|---|---|
| Service category | **Other endpoint services** (not "AWS services") |
| Service name | Paste the service name from Part 1 |
| VPC | Select consumer VPC (e.g., DemoVPC) |
| Subnet | Select subnet for the ENI |

After creation, an ENI is provisioned in the consumer VPC. Traffic to the service flows privately through PrivateLink — no public internet.

---

## Key Console Navigation

| Task | Location |
|---|---|
| Create endpoint service (provider) | VPC → Endpoint Services |
| Create endpoint (consumer) | VPC → Endpoints → Other endpoint services |

---

## Quick Reference

```
PrivateLink Setup:
  Provider: VPC → Endpoint Services → Create → select NLB → note service name
  Consumer: VPC → Endpoints → Create → "Other endpoint services" → paste service name → select VPC

Result: ENI created in consumer VPC → private connection to provider NLB
No public internet, no VPC peering required
```
