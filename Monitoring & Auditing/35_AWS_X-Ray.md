# 35. AWS X-Ray

## Overview

**AWS X-Ray** provides distributed tracing and visual analysis of applications. It is designed for microservices and distributed architectures where combining logs from multiple services makes traditional debugging extremely difficult.

---

## The Problem X-Ray Solves

| Traditional Debugging | With X-Ray |
|----------------------|-----------|
| Add log statements, redeploy, search logs | Visual service map of the entire request flow |
| Logs spread across many services — hard to correlate | Single unified trace per request |
| No common view of distributed architecture | Full picture of each service and its performance |
| Hard to identify which service caused the failure | Pinpoint the exact service or call that failed |

> X-Ray is essential when you have distributed services connected through SQS queues, SNS topics, APIs, or other decoupled communication.

---

## X-Ray Capabilities

| Capability | Detail |
|-----------|--------|
| **Distributed tracing** | Trace a single request end-to-end across all services |
| **Service graph** | Visual map of all services and their dependencies |
| **Performance bottlenecks** | Identify which service is slow or throttling |
| **Error and exception analysis** | See exactly where a request failed and why |
| **SLA monitoring** | Determine if services are responding within acceptable time |
| **Throttling detection** | Identify where requests are being slowed down |
| **User impact analysis** | Determine which users are affected by an outage |

---

## Architecture — How X-Ray Works

```
Request enters your system
        ↓
Passes through Service A → SQS → Service B → DynamoDB
        ↓
X-Ray traces each hop with timing and status
        ↓
X-Ray Console → service graph + per-request trace view
        ↓
Identify: latency, errors, throttling, dependency failures
```

---

## When to Use X-Ray

| Signal | Interpretation |
|--------|---------------|
| "Distributed tracing" | X-Ray |
| "Microservices debugging" | X-Ray |
| "Service graph" or "service map" | X-Ray |
| "Trace a request across multiple services" | X-Ray |
| "Find bottlenecks in a distributed application" | X-Ray |

---

## Best Practices

✓ **Enable X-Ray on all services in a microservice architecture** — partial tracing gives incomplete pictures  
✓ **Use the service graph** — visualizes dependencies that are hard to see in logs alone  
✓ **Filter by error/fault/throttle** — quickly isolates failing segments in a large trace  
✓ **Combine with CloudWatch** — X-Ray for tracing, CloudWatch for metrics and alarms  

---

## SysOps Exam Focus

**Q1: "You have a distributed application using SQS, Lambda, and DynamoDB. Users report intermittent slowness but logs from each service show no obvious errors. What AWS service helps you trace the request flow end-to-end?"**
- A) CloudWatch Logs Insights
- B) AWS X-Ray — provides distributed tracing and a visual service graph across all connected services
- C) AWS Config
- D) CloudTrail
- **Answer: B** — X-Ray correlates traces across distributed services; CloudWatch logs are per-service and hard to correlate manually

**Q2: "What does the X-Ray service graph show?"**
- A) CloudWatch metric trends over time
- B) A visual map of all services in the architecture, their connections, latency, and error rates
- C) A list of IAM policy violations
- D) EC2 instance CPU utilization per availability zone
- **Answer: B** — The service graph is X-Ray's core feature: it visualizes dependencies and highlights performance/error data per service

**Q3: "Which of the following problems is AWS X-Ray best suited to solve?"**
- A) Detecting unauthorized IAM actions
- B) Monitoring EC2 disk utilization
- C) Identifying which microservice in a distributed system is causing request failures or latency
- D) Tracking S3 bucket policy changes
- **Answer: C** — X-Ray is purpose-built for distributed tracing; IAM/S3/EC2 monitoring is handled by CloudTrail and CloudWatch

**Q4: "An application uses SNS to fan out messages to multiple Lambda functions. You want to know if any Lambda function is throttling and impacting end-to-end latency. What should you use?"**
- A) CloudTrail — records all Lambda invocations
- B) AWS X-Ray — traces the full path from SNS through each Lambda, showing throttling and latency per segment
- C) VPC Flow Logs — captures network traffic
- D) CloudWatch Logs — search for throttling error messages in each function's log group
- **Answer: B** — X-Ray shows the complete trace including throttling at each service hop; CloudWatch Logs requires manual correlation across multiple log groups

---

## Quick Reference

```
AWS X-Ray:
  Purpose: distributed tracing for microservices and decoupled architectures
  Core feature: service graph — visual map of all services + latency + errors

  What it solves:
    - Hard-to-correlate logs across multiple services
    - No common view of distributed request flow
    - Unknown bottleneck or failure point

  Capabilities:
    - End-to-end request tracing
    - Bottleneck identification
    - Error and exception pinpointing
    - Throttling detection
    - SLA compliance visibility
    - User impact scope

Exam signals → X-Ray:
  "distributed tracing", "service graph",
  "trace request across services", "microservices debugging"
```

---

**File: 35_AWS_X-Ray.md**
**Status: SysOps-focused, exam-ready, concise format**
