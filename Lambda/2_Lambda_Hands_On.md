# 35 — AWS Lambda Hands-On

## Overview

This lecture walks through creating and testing a Lambda function in the AWS console. Key operational concepts: execution roles, test events, monitoring via CloudWatch, and configuration options.

---

## Creating a Lambda Function

### Options
| Method | Description |
|---|---|
| **Author from scratch** | Write your own code |
| **Blueprint** | Start from a pre-built template (e.g., hello-world) |
| **Container image** | Use a custom Docker image |

### Key Settings at Creation
- **Runtime** — Choose language (Python, Node.js, Java, etc.)
- **Function name** — e.g., `HelloWorld`
- **Execution role** — IAM role the function assumes at runtime

---

## Execution Role

> The execution role for Lambda is equivalent to the **instance profile role** on EC2 — it defines what AWS services the function can access.

### Default Role (Basic Lambda Permissions)
- Automatically created when you select "Create a new role with basic Lambda permissions"
- Grants **CloudWatch Logs** write access only:
  - `logs:CreateLogGroup`
  - `logs:CreateLogStream`
  - `logs:PutLogEvents`

### Adding More Permissions
- To access S3, DynamoDB, etc. — attach additional policies to the execution role
- This is the **only** way to grant a Lambda function AWS resource access (no hardcoded credentials)

---

## Testing a Function

### Test Events
- Create named test events in the console (e.g., `HelloWorldEvent`)
- Test events are **saved** and reusable
- Input is a **JSON payload** passed to the handler

### Example Hello World Handler (Python)
```python
def lambda_handler(event, context):
    return event['key1']  # Returns value of key1 from input JSON
```

### Test Input JSON (hello-world blueprint)
```json
{
  "key1": "value1",
  "key2": "value2",
  "key3": "value3"
}
```

### Test Result
- **Success** → returns `"value1"`
- **Failure** → if `key1` is missing, function throws a `KeyError` — visible in logs

---

## Monitoring

### CloudWatch Integration (automatic)
- Lambda automatically sends logs to **CloudWatch Logs**
- Each function has its own **Log Group**: `/aws/lambda/<function-name>`
- Each execution creates a **Log Stream**

### What You See in Logs
- Invocation start/end timestamps
- Return values
- Errors and stack traces (for failed executions)

### Console Metrics Tab
Shows (after a short delay):
- Invocations count
- Error count
- Duration
- Throttles

---

## Configuration Options

### General Configuration
| Setting | Description |
|---|---|
| **Memory** | 128 MB to 10,240 MB (10 GB); also affects CPU/network |
| **Ephemeral Storage** | Temporary `/tmp` space (512 MB to 10 GB) |
| **Timeout** | How long before Lambda forcibly stops execution (max 15 min) |
| **Execution Role** | IAM role used at runtime |

### Triggers
- Add event sources directly from the console
- Common triggers visible in console:
  - Amazon S3 (object created/removed events)
  - API Gateway
  - SQS, SNS, DynamoDB Streams, Kinesis
  - EventBridge (scheduled or event-based)
  - Cognito, CloudFront, and many more

---

## Scaling Behaviour (Demo Observation)

- As more events are sent simultaneously, Lambda **automatically scales up** concurrent executions
- Each concurrent invocation = a separate function instance (shown as separate "cogs" in the console demo)
- No manual intervention needed — AWS handles provisioning

---

## Best Practices

- Always use the **execution role** for AWS API access — never hardcode credentials
- Use **saved test events** for repeatable testing during development
- Check **CloudWatch Logs** first when debugging failures
- Set an appropriate **timeout** — default is 3 seconds; increase for longer tasks but keep well under 15 min
- Size **memory** based on workload — RAM increase also boosts CPU and network

---

## SysOps Exam Q&A

**Q: A Lambda function needs to write objects to S3. What must be configured?**
A: Add an **IAM policy** granting S3 write permissions to the function's **execution role**.

**Q: Where do Lambda function logs appear by default?**
A: **Amazon CloudWatch Logs**, in a log group named `/aws/lambda/<function-name>`.

**Q: A Lambda function times out before completing. What should you adjust?**
A: Increase the **timeout** setting in General Configuration (max 15 minutes) and/or increase **memory** if the task is CPU-bound.

**Q: How does Lambda handle a sudden spike in concurrent invocations?**
A: Lambda **automatically scales** by spinning up additional concurrent execution environments — no manual scaling configuration required.

**Q: What is the execution role in Lambda analogous to on EC2?**
A: The **EC2 instance profile (IAM role)** — it grants the compute resource permissions to call AWS services.

---

## Quick Reference

```
Execution role  = IAM role assumed by Lambda at runtime (like EC2 instance profile)
Default perms   = CloudWatch Logs only (write logs)
Add AWS access  = attach policies to execution role
Test events     = saved JSON payloads, reusable in console
Logs location   = CloudWatch Logs → /aws/lambda/<function-name>
Scaling         = automatic, no configuration needed
Timeout default = 3 seconds (max 15 minutes)
Memory range    = 128 MB – 10,240 MB
```
