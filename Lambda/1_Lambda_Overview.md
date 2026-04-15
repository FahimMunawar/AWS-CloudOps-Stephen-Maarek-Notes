# 34 — AWS Lambda Overview

## Overview

AWS Lambda is a **serverless compute service** where you provide code and AWS runs it — no servers to provision, patch, or manage. Functions execute on-demand and billing is purely usage-based, making it a fundamental shift from the EC2 model.

---

## EC2 vs Lambda Comparison

| Dimension | Amazon EC2 | AWS Lambda |
|---|---|---|
| Server management | You provision & manage | AWS manages everything |
| Runtime model | Continuously running | On-demand (runs only when invoked) |
| Billing | Per second (even when idle) | Per request + compute duration |
| Scaling | Auto Scaling Groups (you configure) | Automatic, built-in concurrency scaling |
| Max execution time | Unlimited | **15 minutes** per invocation |

---

## Key Characteristics

- **Virtual functions** — no servers to manage
- **Short executions** — maximum **15 minutes** per invocation
- **On-demand** — not running (and not billed) when idle
- **Auto-scaling** — AWS automatically provisions concurrent executions as needed

---

## Supported Runtimes

| Language | Notes |
|---|---|
| Node.js (JavaScript) | Most popular |
| Python | Most popular |
| Java | |
| C# (.NET Core) | |
| PowerShell | |
| Ruby | |
| Custom Runtime API | Supports Rust, Go, and others |
| Container images | Must implement Lambda Runtime API |

> **Exam tip:** For running Docker/container images, prefer **ECS or Fargate** over Lambda. Lambda does support custom container images (via Runtime API), but ECS/Fargate is the standard answer for containers on the exam.

---

## Pricing Model

### Per Requests (Invocations)
- First **1,000,000 requests/month** — free
- After that: **$0.20 per million requests**

### Per Duration (Compute Time)
- Billed in **1-millisecond increments**
- Free tier: **400,000 GB-seconds/month**
  - = 400,000 seconds at 1 GB RAM
  - = 3,200,000 seconds at 128 MB RAM (8× more seconds)
- After free tier: **$1.00 per 600,000 GB-seconds**

> **RAM matters for pricing and performance:** Increasing RAM also improves CPU and network performance proportionally.

---

## Resource Limits

| Resource | Limit |
|---|---|
| Max RAM per function | **10 GB** |
| Max execution timeout | **15 minutes** |
| Increasing RAM effect | Also improves CPU & network performance |

---

## Key AWS Integrations

| Service | How Lambda is Used |
|---|---|
| **API Gateway** | Creates REST APIs that invoke Lambda functions |
| **S3** | Lambda triggered on file upload/change events |
| **DynamoDB** | Triggers on table item changes (Streams) |
| **Kinesis** | Real-time data transformation |
| **CloudWatch Events / EventBridge** | React to AWS infrastructure events, scheduled tasks |
| **CloudWatch Logs** | Stream logs to destinations |
| **SNS** | React to topic notifications |
| **SQS** | Process messages from queues |
| **CloudFront** | Lambda@Edge — run functions at CDN edge locations |
| **Cognito** | React to user authentication events |

---

## Real-World Architecture Examples

### 1 — Serverless Thumbnail Creation

```
User uploads image
       │
       ▼
  S3 Bucket (original)
       │  S3 Event Notification
       ▼
  Lambda Function
  ├── Generates thumbnail → S3 Bucket (thumbnails)
  └── Inserts metadata (name, size, date) → DynamoDB
```

- Fully event-driven: Lambda only runs when a new image arrives
- No servers to maintain

### 2 — Serverless CRON Job

```
EventBridge Rule (e.g., every 1 hour)
       │
       ▼
  Lambda Function (performs scheduled task)
```

- Replaces EC2-hosted CRON jobs
- Both EventBridge and Lambda are serverless — no idle compute cost
- EC2-based CRON wastes compute time between executions

---

## Best Practices

- Use Lambda for **event-driven**, **short-duration** tasks (under 15 min)
- For long-running processes, use EC2, ECS/Fargate, or AWS Batch
- Size RAM appropriately — RAM increase directly improves CPU/network
- Pair with EventBridge for scheduled (CRON-style) serverless automation
- Use SQS/Kinesis as event sources for decoupled, scalable pipelines

---

## SysOps Exam Q&A

**Q: What is the maximum execution timeout for a Lambda function?**
A: **15 minutes**

**Q: You need to run a Docker container on AWS. Should you use Lambda or ECS?**
A: **ECS or Fargate** — Lambda supports custom containers but ECS/Fargate is the standard exam answer for Docker workloads.

**Q: How does increasing a Lambda function's RAM affect performance?**
A: Increasing RAM also **proportionally improves CPU and network** performance.

**Q: What is the Lambda free tier?**
A: **1 million requests/month** and **400,000 GB-seconds of compute time/month**.

**Q: A company wants to trigger processing whenever a file is uploaded to S3. What is the recommended architecture?**
A: **S3 Event Notification → Lambda function** — fully serverless, no polling required.

**Q: How does Lambda billing differ from EC2?**
A: Lambda charges only when the function is **actively running** (per request + per duration). EC2 charges continuously whether or not workloads are running.

---

## Quick Reference

```
Lambda = serverless functions, on-demand, auto-scaling
Max timeout    = 15 minutes
Max RAM        = 10 GB (also improves CPU/network)
Billing        = per request + per GB-second of duration
Free tier      = 1M requests + 400K GB-seconds/month
Containers     = supported, but prefer ECS/Fargate for Docker (exam)
Best triggers  = S3, API Gateway, SQS, EventBridge, DynamoDB Streams
```
