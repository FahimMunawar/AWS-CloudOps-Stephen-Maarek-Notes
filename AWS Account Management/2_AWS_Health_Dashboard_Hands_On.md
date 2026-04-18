# 2 — AWS Health Dashboard: Hands-On

## Navigation

Console → top-right **bell icon** → **Event Log** → Health Dashboard

---

## Service Health Tab

- **Service History**: health of all AWS services, by region, by day
- Browse by region (e.g., North America) and service (e.g., Amazon EventBridge Scheduler)
- Filter by specific service or specific region
- Shows **open issues** affecting AWS services globally (not personalized)

---

## Account Health Tabs

| Tab | Content |
|---|---|
| **Open and Recent Issues** | Issues currently impacting your account; bell icon shows a badge when active |
| **Scheduled Changes** | Upcoming maintenance (e.g., EBS volume maintenance) that may affect your resources |
| **Other Notifications** | Miscellaneous alerts relevant to your account |
| **Event Log** | Full history of opened + closed issues, with start time and last update time |

### Event Log Detail View

Click any event to see:
- Start time and resolution time
- Event description
- Impact details
- **Affected Resources** tab — exact resources impacted in your account

---

## Organization Health

- Configure organization-wide visibility for all member accounts
- Requires enabling in the Health Dashboard settings

---

## EventBridge Integration

Health events can trigger **Amazon EventBridge rules** for automated responses (e.g., notify on-call, run SSM automation).

```
AWS Health event → EventBridge rule → Lambda / SNS / SSM Automation
```

---

## Quick Reference

```
Access: console bell icon → Event Log

Service Health   → all services, all regions, by day (general)
Account Health   → only events impacting YOUR resources
  Open Issues    → active right now (bell badge when present)
  Scheduled      → upcoming maintenance
  Event Log      → full history with affected resources

Organization Health → aggregated view across all accounts
EventBridge         → automate responses to health events
```
