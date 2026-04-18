# 16 — Amazon EventBridge

## Overview

Amazon EventBridge (formerly **CloudWatch Events**) is a serverless event bus that routes events from AWS services, SaaS partners, and custom applications to various destinations. It supports both **event-driven** and **scheduled (cron)** patterns.

---

## Event Buses

| Bus Type | Source | Notes |
|---|---|---|
| **Default event bus** | AWS services | EC2, S3, CodeBuild, Trusted Advisor, CloudTrail, etc. |
| **Partner event bus** | SaaS partners | Zendesk, Datadog, Auth0, and others send events directly |
| **Custom event bus** | Your own applications | Send your own events; accessible cross-account via resource-based policy |

---

## Event Sources (Examples)

| Source | Event Example |
|---|---|
| EC2 | Instance started / stopped / terminated |
| S3 | Object uploaded |
| CodeBuild | Build failed |
| Trusted Advisor | New security finding |
| CloudTrail + EventBridge | **Any API call** made in the AWS account |
| Schedule / Cron | Every 4 hours, every Monday at 8am, first Monday of month |

> **Exam tip:** Combining CloudTrail + EventBridge lets you intercept **any API call** in your account.

---

## Event Flow

```
Event Source (EC2, S3, CloudTrail, Schedule, SaaS partner, custom app)
       │
       ▼
Amazon EventBridge (event bus)
       │
       Filter rule (optional — e.g., only events for a specific S3 bucket)
       │
       ▼
JSON event document (contains: resource ID, time, IP, details, etc.)
       │
       ▼
Destination (action)
```

---

## Destinations (Targets)

| Destination | Use Case |
|---|---|
| **Lambda** | Custom logic / processing |
| **SQS / SNS** | Messaging and notifications |
| **Kinesis Data Stream** | Stream processing |
| **Step Functions** | Orchestrate workflows |
| **ECS Task** | Launch containers |
| **AWS Batch** | Run batch jobs |
| **CodePipeline / CodeBuild** | Trigger CI/CD |
| **SSM Automation** | Run Systems Manager automations |
| **EC2 Actions** | Start / stop / reboot instances |

---

## Key Features

### Schema Registry
- EventBridge analyzes events in your bus and **infers their schema**
- Schemas are versioned
- Download **generated code** for your application so it knows the event data structure in advance

### Event Archiving & Replay
- Archive **all or filtered** events (indefinite or set retention period)
- **Replay archived events** — useful for debugging Lambda bugs or re-processing after a fix

### Resource-Based Policies
- Control which accounts/regions can send events to your event bus
- Use case: **central event bus** in one account within an AWS Organization — other accounts `PutEvents` into it

---

## Common Exam Pattern — Root User Alert

```
IAM root user sign in (event)
  → EventBridge rule
    → SNS topic
      → Email notification
```

---

## SysOps Exam Q&A

**Q: What was Amazon EventBridge previously called?**
A: **CloudWatch Events** — same service, renamed.

**Q: How do you intercept any API call made in an AWS account using EventBridge?**
A: Combine **CloudTrail + EventBridge** — CloudTrail captures every API call and EventBridge rules can react to them.

**Q: What are the three types of EventBridge event buses?**
A: **Default** (AWS services), **Partner** (SaaS integrations like Datadog/Zendesk), **Custom** (your own applications).

**Q: How would you alert on root account logins using EventBridge?**
A: Create an EventBridge rule matching IAM root user sign-in events → target SNS topic → email notification.

**Q: What is the EventBridge Schema Registry?**
A: It analyzes events on the bus, infers their structure, and lets you download generated code so your application knows the event format in advance. Schemas are versioned.

**Q: How do you aggregate events from multiple AWS accounts into one central event bus?**
A: Add a **resource-based policy** on the central event bus allowing other accounts to call `PutEvents` on it.

---

## Quick Reference

```
EventBridge = formerly CloudWatch Events
Event buses: Default (AWS) | Partner (SaaS) | Custom (your apps)

Sources: EC2, S3, CodeBuild, Trusted Advisor, Schedule/cron, CloudTrail (any API call), SaaS

Targets: Lambda, SQS, SNS, Kinesis, Step Functions, ECS, Batch, CodePipeline, SSM, EC2 actions

Schema Registry:
  Infers event schema → versioned → downloadable generated code

Archive + Replay:
  Archive all/filtered events → replay for debugging

Resource-based policy:
  Allow cross-account PutEvents → central event bus pattern
```
