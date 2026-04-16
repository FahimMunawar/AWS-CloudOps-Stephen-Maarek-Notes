# 1 — Amazon S3 Overview

## Overview

Amazon S3 (Simple Storage Service) is one of AWS's core building blocks — advertised as **infinitely scalable storage**. Much of the web and many AWS services rely on S3 for storage and integrations.

---

## Use Cases

| Use Case | Description |
|---|---|
| **Backup & Storage** | Files, disk backups, and general-purpose storage |
| **Disaster Recovery** | Move data to another region as a backup |
| **Archival** | Long-term archive with retrieval at low cost (S3 Glacier) |
| **Hybrid Cloud Storage** | Extend on-premises storage into the cloud |
| **Application Hosting** | Host application assets |
| **Media Hosting** | Videos, images, and other static content |
| **Data Lake / Analytics** | Store massive datasets for big data analytics |
| **Software Delivery** | Host software update packages |
| **Static Website Hosting** | Serve HTML/CSS/JS directly from S3 |

### Real-World Examples
- **Nasdaq** — stores 7 years of data in S3 Glacier
- **Sysco** — runs analytics on S3 data for business insights

---

## Buckets

- S3 stores files in **buckets** (top-level directories)
- Buckets are **defined at the region level** — despite S3 appearing global in the console, each bucket exists in a specific region
- Bucket names must be **globally unique** across all regions and all AWS accounts

> **Common misconception:** S3 looks like a global service but buckets are regional. This is a frequent exam trap.

### Bucket Naming Rules
- No uppercase letters
- No underscores
- 3–63 characters long
- Must not be formatted as an IP address
- Must start with a lowercase letter or number
- Use only: lowercase letters, numbers, hyphens

---

## Objects

Objects are the files stored in buckets. Every object has a **key**.

### Object Key
The key is the **full path** to the object within the bucket.

| Example | Key |
|---|---|
| File at root | `my-file.txt` |
| Nested file | `my-folder/another-folder/my-file.txt` |

### Key = Prefix + Object Name

```
my-folder/another-folder/my-file.txt
│                        │
└── Prefix ──────────────┘└── Object Name ──┘
    my-folder/another-folder/               my-file.txt
```

> **Important:** S3 has **no real concept of directories**. The console shows folders, but everything is actually a flat key. Folders are just a visual representation of key prefixes containing `/`.

### Object Properties

| Property | Details |
|---|---|
| **Value** | The content/body of the file (anything you upload) |
| **Max object size** | **5 TB** (5,000 GB) |
| **Multi-part upload** | Required for files **> 5 GB**; splits into multiple parts |
| **Metadata** | Key-value pairs (system or user-defined); describes the object |
| **Tags** | Unicode key-value pairs, up to **10 per object**; used for security and lifecycle policies |
| **Version ID** | Present if versioning is enabled on the bucket |

### Multi-Part Upload Rule
- File > 5 GB → **must** use multi-part upload
- Example: 5 TB file = at least 1,000 parts of 5 GB each

---

## Key Facts Summary

```
Buckets     = regional (not global, despite global-looking console)
Bucket name = globally unique across all regions and all accounts
Objects     = files stored in buckets, identified by a key
Key         = full path = prefix + object name
No real directories — keys with "/" just look like folders
Max size    = 5 TB per object
> 5 GB      = must use multi-part upload
Tags        = up to 10 per object (security + lifecycle use)
```

---

## SysOps Exam Q&A

**Q: Are S3 bucket names unique per region or globally?**
A: **Globally unique** — across all regions and all AWS accounts.

**Q: Are S3 buckets global or regional?**
A: **Regional** — each bucket is created in a specific region, even though the S3 console appears global.

**Q: What is an S3 object key?**
A: The **full path** to the object in the bucket, composed of a prefix (folder path) and the object name.

**Q: Does S3 have real directories/folders?**
A: **No** — S3 uses a flat key namespace. Folders are a UI representation of key prefixes containing `/`.

**Q: What is the maximum size of a single S3 object?**
A: **5 TB** (5,000 GB).

**Q: When is multi-part upload required?**
A: For any file **larger than 5 GB**. It is recommended for files over 100 MB.

**Q: What are S3 object tags used for?**
A: **Security** (IAM/bucket policy conditions) and **lifecycle management**. Up to 10 tags per object.
