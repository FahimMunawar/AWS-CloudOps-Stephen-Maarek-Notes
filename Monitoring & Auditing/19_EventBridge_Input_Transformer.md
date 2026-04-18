# 19 — EventBridge Input Transformer

## Overview

Input Transformers let you reshape an EventBridge event before sending it to a target — extract specific fields, rename them, and build a custom output message.

---

## Why Use It

A downstream target (e.g., CloudWatch Logs, Lambda) may not want the full raw event JSON. Input transformers let you:
- Extract only the fields you need
- Rename or restructure fields
- Build a human-readable message string

---

## How It Works

```
EventBridge Rule matches event
  └── Additional settings → Configure target input → Input Transformer
        │
        ├── Input Path: define variables by extracting fields from the event JSON
        └── Input Template: use those variables to build the output sent to the target
```

---

## Input Path Syntax

Variables are defined as a JSON object. Use `$.` to navigate the event JSON:

| Syntax | Meaning |
|---|---|
| `$.time` | Top-level `time` field |
| `$.detail.instance-id` | Nested field under `detail` |
| `$.resources[0]` | First element of the `resources` array |

### Example Input Path (EC2 State-Change Event)

```json
{
  "timestamp": "$.time",
  "instance":  "$.detail.instance-id",
  "state":     "$.detail.state",
  "resource":  "$.resources[0]"
}
```

- Up to **100 variables** can be defined

---

## Input Template

Use variable names wrapped in `<variable>` to build the output:

```json
{
  "timestamp": "<timestamp>",
  "message": "Instance <instance> is in state <state> at <timestamp> with ARN <resource>"
}
```

### Generated Output (example)

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "message": "Instance i-0abc123 is in state pending at 2024-01-15T10:30:00Z with ARN arn:aws:ec2:..."
}
```

> CloudWatch Log Groups require the output to be a **JSON document** (not a plain string).

---

## Demo Flow

```
EC2 instance launched
  → EventBridge rule (EC2 Instance State-change Notification)
    → Input Transformer extracts: timestamp, instance-id, state, ARN
      → CloudWatch Log Group (/aws/events/event-input-transformation)
        → Log stream shows: "Instance i-xxx is in state pending at <timestamp>"
```

---

## Where to Configure (Console)

1. Create rule → select event source → **Next**
2. Select target (e.g., CloudWatch log group)
3. Expand **Additional settings**
4. Under **Configure target input** → select **Input transformer**
5. Define **Input path** (variables) and **Input template** (output shape)
6. Use **Generate output** to preview before saving

---

## Quick Reference

```
Input Path:    $.fieldName or $.nested.field or $.array[0]
               Up to 100 variables
Input Template: use <variableName> to reference variables

CloudWatch Logs target: output must be a JSON document, not a plain string

Use case: reshape raw EventBridge event into a simplified or custom format
          before sending to target
```
