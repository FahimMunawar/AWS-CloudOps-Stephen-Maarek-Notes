# 8 — S3 Versioning: Propagation Delay Note

## Overview

A specific exam-targeted fact about enabling S3 versioning for the first time on a bucket.

---

## The Rule

> When you **enable versioning on a bucket for the first time**, it may take up to **15 minutes** for the change to fully propagate.

During this propagation window, attempting to **get an object that was just created or updated** may return:

```
HTTP 404 NoSuchKey
```

Even though the object exists, S3 hasn't fully propagated the versioning state yet.

---

## What To Do

- Wait **at least 15 minutes** after enabling versioning before issuing write or read operations on that bucket
- If you see a `404 NoSuchKey` error immediately after enabling versioning, this is likely the cause

---

## SysOps Exam Q&A

**Q: You enable versioning on an S3 bucket and immediately try to read an object. You get a 404 NoSuchKey error. What is the most likely cause?**
A: Versioning was just enabled and the change **has not fully propagated yet**. Wait 15 minutes before issuing operations.

**Q: How long should you wait after enabling S3 versioning before performing write operations?**
A: **At least 15 minutes** for the propagation to complete.

---

## Quick Reference

```
Enable versioning (first time) → wait 15 minutes
During propagation             → 404 NoSuchKey errors possible
After 15 minutes               → safe to read/write
```
