# 4 — S3 Event Notifications — Hands-On

## Overview

Console walkthrough: configure an S3 Event Notification to send object creation events to an SQS queue, including fixing the access policy error.

---

## Two Ways to Configure Event Notifications (Console)

S3 → Bucket → **Properties** → scroll to **Event notifications**:

| Option | How |
|---|---|
| **Create event notification** | Direct to SNS / SQS / Lambda |
| **Enable EventBridge integration** | Toggle ON — all events sent to EventBridge automatically |

---

## Hands-On: S3 → SQS Event Notification

### Step 1 — Create an SQS Queue
- Go to Amazon SQS → **Create queue**
- Name: `DemoS3Notification`
- Leave defaults → Create

### Step 2 — Configure the Event Notification in S3
- S3 → Bucket → Properties → Event notifications → **Create event notification**
- Name: `DemoEventNotification`
- Prefix/Suffix: optional (leave blank to catch all objects)
- **Event types**: All object create events (or select specific ones)
- **Destination**: SQS queue → select `DemoS3Notification`
- Click Save → **Error!**

### Step 3 — Fix the SQS Access Policy (the key step)

The error message: *"Cannot validate destination configuration"*

**Cause:** The SQS queue does not have a policy allowing S3 to send messages to it.

**Fix:**
1. Go to SQS → `DemoS3Notification` → **Edit** → **Access policy**
2. Use the **Policy Generator** to create an SQS Queue Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "sqs:SendMessage",
      "Resource": "arn:aws:sqs:<region>:<account-id>:DemoS3Notification"
    }
  ]
}
```

3. Save the access policy

### Step 4 — Retry Saving the Event Notification
- Go back to S3 → Event notification → Save changes
- Now succeeds

### Step 5 — Verify the Test Message
When the event notification is first saved, **S3 automatically sends a test connectivity message** to the SQS queue.

- SQS → `DemoS3Notification` → **Send and receive messages** → **Poll for messages**
- A test message appears — confirms the connection works
- Delete the test message

---

## Testing: Upload an Object

1. Upload `coffee.jpg` to the S3 bucket
2. Go to SQS → Poll for messages
3. A new message appears with:

```json
{
  "eventName": "ObjectCreated:Put",
  "s3": {
    "bucket": { "name": "stephane-v3-events-notifications" },
    "object": { "key": "coffee.jpg" }
  }
}
```

- `eventName: ObjectCreated:Put` — confirms the upload triggered the event
- `object.key: coffee.jpg` — identifies the uploaded file
- This message can now be consumed by a worker to process the file (e.g., generate a thumbnail)

---

## Key Points from Hands-On

| Point | Detail |
|---|---|
| **Error on save** | SQS access policy must allow `sqs:SendMessage` from S3 before the notification can be saved |
| **Test message** | S3 sends a connectivity test message to the queue automatically when notification is first saved |
| **Event payload** | Contains `eventName`, `bucket.name`, `object.key` — enough to retrieve and process the object |
| **EventBridge toggle** | Simple on/off to send ALL events to EventBridge — no per-event configuration needed |

---

## SysOps Exam Q&A

**Q: You configure an S3 Event Notification targeting SQS but get a validation error when saving. What is the fix?**
A: Update the **SQS access policy** to allow `sqs:SendMessage` from the S3 service (`s3.amazonaws.com`) or the specific bucket ARN.

**Q: After saving an S3 Event Notification, a message immediately appears in the SQS queue. What is it?**
A: A **test connectivity message** that S3 sends automatically to verify the destination is reachable and the permissions are correct.

**Q: What information is in the S3 event notification payload sent to SQS?**
A: `eventName` (e.g., `ObjectCreated:Put`), `bucket.name`, and `object.key` — enough to identify and retrieve the affected object.

---

## Quick Reference

```
Setup order:
  1. Create SQS queue
  2. Add SQS access policy (sqs:SendMessage, Principal: *)
  3. Create S3 event notification → select SQS queue
  4. Save → auto test message sent to queue
  5. Upload file → event message appears in queue

EventBridge: toggle ON in same section → no per-event config needed
Error cause: missing sqs:SendMessage in SQS access policy
Test message: auto-sent by S3 when notification is first saved
```
