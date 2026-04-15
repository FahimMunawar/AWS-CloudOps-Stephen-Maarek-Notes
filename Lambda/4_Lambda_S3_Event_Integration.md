# 4 — Lambda + S3 Event Notifications Integration

## Overview

S3 Event Notifications allow S3 to automatically invoke a Lambda function (or SNS/SQS) whenever objects are created, removed, or replicated. This is an **asynchronous invocation** pattern.

---

## S3 Event Notification Recap

- Triggered on: object **created**, **removed**, **restored**, **replication**
- Can filter by **prefix** (e.g., `images/`) and **suffix** (e.g., `.jpg`)
- Classic use case: generate thumbnails for every image uploaded to S3

---

## Three Destination Options

```
S3 Event
   │
   ├──► SNS Topic ──► Fan-out to multiple SQS queues
   │
   ├──► SQS Queue ──► Lambda reads from SQS (poll-based)
   │
   └──► Lambda Function (direct, asynchronous invocation)
                   └──► On failure: Dead-Letter Queue (SQS)
```

| Destination | Pattern |
|---|---|
| **SNS** | Fan-out — broadcast to multiple subscribers |
| **SQS → Lambda** | Decoupled queue; Lambda polls SQS |
| **Lambda (direct)** | Async invocation; fastest path |

---

## Important: Event Delivery Guarantee

- Events typically delivered in **seconds**, but can take **up to a minute or longer**
- **Enable versioning on your S3 bucket** to avoid missing events
  - If two writes to the **same object key** happen simultaneously without versioning → you may only get **one notification** instead of two

---

## Simple Architecture Pattern

```
S3 Bucket (file upload)
       │  S3 Event Notification
       ▼
  Lambda Function
  ├── Insert data → DynamoDB Table
  └── Insert data → RDS Database
```

---

## How S3 Gets Permission to Invoke Lambda: Resource-Based Policy

When you configure an S3 event notification targeting a Lambda function, AWS automatically adds a **resource-based policy** to the Lambda function.

### Policy Example
```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "s3.amazonaws.com"
  },
  "Action": "lambda:InvokeFunction",
  "Resource": "arn:aws:lambda:<region>:<account-id>:function:lambda-s3",
  "Condition": {
    "ArnLike": {
      "AWS:SourceArn": "arn:aws:s3:::demo-s3-events-bucket"
    }
  }
}
```

### Key Points
- **Principal**: `s3.amazonaws.com` — S3 service is granted invoke permission
- **Condition**: `ArnLike` on `SourceArn` — only **this specific bucket** can invoke the function
- Configured automatically when you set Lambda as the event notification destination in the S3 console

---

## Event Payload Sent to Lambda

When S3 triggers Lambda, the event object contains details about the object:

```json
{
  "Records": [
    {
      "eventSource": "aws:s3",
      "awsRegion": "eu-west-1",
      "eventName": "ObjectCreated:Put",
      "s3": {
        "bucket": {
          "name": "demo-s3-events-bucket",
          "arn": "arn:aws:s3:::demo-s3-events-bucket"
        },
        "object": {
          "key": "coffee.jpeg",
          "size": 12345,
          "eTag": "abc123..."
        }
      }
    }
  ]
}
```

### Using the Payload in Code
```python
def lambda_handler(event, context):
    print(event)  # Full S3 event JSON logged to CloudWatch
    
    bucket = event['Records'][0]['s3']['bucket']['name']
    key    = event['Records'][0]['s3']['object']['key']
    
    # Now use S3 GetObject API to retrieve and process the file
    # s3_client.get_object(Bucket=bucket, Key=key)
```

The Lambda function uses `bucket` + `key` from the event to call `s3:GetObject` and process the uploaded file.

---

## Hands-On Steps

1. Create Lambda function (e.g., `lambda-s3`, Python 3.8)
2. Create S3 bucket in the **same region** as the Lambda function
3. S3 → Bucket → Properties → **Event notifications** → Create event notification
   - Name: `invoke-lambda`
   - Prefix/Suffix: leave blank for all objects
   - Event types: **All object create events**
   - Destination: **Lambda function** → select `lambda-s3`
4. Refresh Lambda console → **Triggers** tab now shows S3 as source
5. Verify resource-based policy: Lambda → Configuration → **Permissions** → Resource-based policy statements
6. Upload a test file to S3 → check Lambda → Monitor → **View CloudWatch Logs**

---

## Monitoring & Debugging

- Lambda → Monitor → View CloudWatch Logs
- Each S3 upload creates a new log stream entry
- Event payload in logs shows: `eventSource`, `bucket.name`, `object.key`, `object.size`, `eTag`
- Use `object.key` in code to call `s3:GetObject` and process the file

---

## Best Practices

- **Same region**: S3 bucket and Lambda function must be in the same AWS region
- **Enable versioning**: prevents missing notifications on concurrent writes to the same key
- **Use Dead-Letter Queue (SQS)**: capture failed async invocations for retry/debugging
- **Least privilege**: Lambda's execution role only needs `s3:GetObject` on the specific bucket, not broad S3 access
- For high-throughput uploads, consider **SQS → Lambda** pattern instead of direct S3 → Lambda to avoid throttling

---

## SysOps Exam Q&A

**Q: How does S3 get permission to invoke a Lambda function?**
A: AWS automatically adds a **resource-based policy** to Lambda granting `lambda:InvokeFunction` to `s3.amazonaws.com`, scoped to the specific bucket ARN via an `ArnLike` condition.

**Q: S3 event notifications are missing some events for the same object. What is the likely cause?**
A: **Versioning is not enabled** on the bucket. Concurrent writes to the same key without versioning can result in only one notification being delivered.

**Q: What information does the S3 event payload provide to Lambda?**
A: `bucket.name`, `object.key`, `object.size`, `eTag`, `eventName`, `awsRegion`, and `eventSource`.

**Q: An S3 event directly invokes Lambda. What type of invocation is this?**
A: **Asynchronous invocation** — S3 fires and forgets; Lambda processes the event independently.

**Q: How can you ensure failed S3 → Lambda invocations are not lost?**
A: Configure a **Dead-Letter Queue (SQS or SNS)** on the Lambda function to capture failed async invocations.

**Q: What are the three destinations for S3 event notifications?**
A: **SNS** (fan-out), **SQS** (queue-based decoupling), and **Lambda** (direct async invocation).

---

## Quick Reference

```
Trigger type     = asynchronous invocation
Permission model = resource-based policy on Lambda (auto-added)
Principal        = s3.amazonaws.com
Condition        = ArnLike on specific bucket ARN
Event payload    = Records[0].s3.bucket.name + object.key
Missing events?  = enable bucket versioning
Failed events?   = configure Dead-Letter Queue on Lambda
Same region      = S3 bucket and Lambda must be in same region
```
