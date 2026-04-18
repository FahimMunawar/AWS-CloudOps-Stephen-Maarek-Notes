# 18 — EventBridge Content Filtering

## Overview

EventBridge event patterns support advanced filtering so rules only trigger on events that match specific conditions — not every event from a source.

---

## Filter Types

| Filter | Description | Example |
|---|---|---|
| **Exact match** | Field equals a specific value | `"source": ["aws.s3"]` |
| **Prefix matching** | Field starts with a string | bucket name starts with `mybuckets` |
| **Suffix matching** | Field ends with a string | object key ends with `.png` |
| **Anything but** | Match all values except specified | anything but `initializing` |
| **Numeric matching** | Numeric comparisons on a field | value > 0 and < 5 |
| **IP address matching** | Match on CIDR range | `"cidr": "192.168.0.0/24"` |
| **Exists matching** | Assert a field is present in the event | `size` field must exist |
| **Equals-ignore-case** | Case-insensitive string match | |

---

## Event Pattern JSON Structure

EventBridge patterns match on the event's top-level fields:

```json
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"],
  "detail": {
    "bucket": {
      "name": [{ "prefix": "mybuckets" }]
    },
    "object": {
      "key": [{ "suffix": ".png" }]
    }
  }
}
```

This pattern matches: S3 Object Created events where the bucket name **starts with** `mybuckets` AND the object key **ends with** `.png`.

---

## Field Paths in S3 Object Created Events

| Data | JSON path |
|---|---|
| Bucket name | `detail.bucket.name` |
| Object key | `detail.object.key` |
| Source IP address | `detail.sourceIPaddress` |
| Object size | `detail.object.size` |

---

## Filter Syntax Examples

```json
// Prefix match
{ "prefix": "mybuckets" }

// Suffix match
{ "suffix": ".png" }

// Anything but
{ "anything-but": "initializing" }

// Numeric match (range)
[{ "numeric": [">", 0, "<", 5] }]

// IP address (CIDR)
[{ "cidr": "10.0.0.0/8" }]

// Exists
[{ "exists": true }]
```

---

## How to Edit the Pattern (Console)

1. Create rule → select event source (e.g., S3) and event type
2. If the dropdown doesn't have your exact event → click **Edit pattern**
3. Modify the JSON directly to add advanced filters
4. Use AWS documentation (search "EventBridge content filtering") for syntax reference

---

## SysOps Exam Q&A

**Q: How do you match S3 events only for buckets whose names start with "prod"?**
A: Use **prefix matching** in the event pattern: `"name": [{ "prefix": "prod" }]` under `detail.bucket`.

**Q: How do you match only `.jpg` file upload events in EventBridge?**
A: Use **suffix matching** on the object key: `"key": [{ "suffix": ".jpg" }]` under `detail.object`.

**Q: How do you ensure an EventBridge rule only fires when a specific field exists in the event?**
A: Use **exists matching**: `[{ "exists": true }]` on the field path.

---

## Quick Reference

```
Edit pattern → JSON form → use filter types:
  Exact:       "field": ["value"]
  Prefix:      "field": [{ "prefix": "abc" }]
  Suffix:      "field": [{ "suffix": ".png" }]
  Anything-but:"field": [{ "anything-but": "x" }]
  Numeric:     "field": [{ "numeric": [">", 0] }]
  CIDR:        "field": [{ "cidr": "10.0.0.0/8" }]
  Exists:      "field": [{ "exists": true }]

S3 Object Created paths:
  detail.bucket.name       → bucket name
  detail.object.key        → object key / file path
  detail.sourceIPaddress   → source IP of the upload
```
