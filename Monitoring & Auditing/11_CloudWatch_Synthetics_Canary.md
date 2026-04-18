# 11 — CloudWatch Synthetics Canary

## Overview

CloudWatch Synthetics Canary runs configurable scripts that programmatically reproduce what your customers do — monitoring APIs, URLs, and web flows **before your customers find issues**.

---

## What It Can Do

- Monitor **availability and latency** of endpoints
- Reproduce **customer journeys** (e.g., add to cart → checkout → payment)
- Store **load time data** and take **UI screenshots**
- Run **once** or on a **regular schedule**

---

## How It Works

```
Synthetics Canary Script (Node.js / Python + headless Chrome)
  └── Runs on schedule → tests your application
        │
        ├── Script passes → OK
        └── Script fails → CloudWatch Alarm → Lambda → action
                                   e.g., update Route 53 DNS
                                   to redirect traffic to healthy region
```

### Example Architecture

```
Application in us-east-1
  └── Synthetics Canary monitors it
        └── Fails → CloudWatch Alarm
              └── Lambda updates Route 53 DNS
                    └── Traffic redirected to us-west-2 (healthy instance)
```

---

## Script Runtime

| Detail | Value |
|---|---|
| **Languages** | Node.js or Python |
| **Browser** | Headless Google Chrome (full browser access) |
| **Schedule** | One-time or recurring |

---

## Canary Blueprints

| Blueprint | Purpose |
|---|---|
| **Heartbeat Monitor** | Load a URL, store screenshots + HTTP archive, verify availability |
| **API Canary** | Test basic read/write functions of REST APIs |
| **Broken Link Checker** | Check all links inside a URL — flag any broken links |
| **Visual Monitoring** | Compare screenshot from canary run against a baseline screenshot |
| **Canary Recorder** | Record browser actions on a website → auto-generate a reusable script |
| **GUI Workflow Builder** | Verify webpage interactions (e.g., login form, multi-step flows) |

---

## SysOps Exam Q&A

**Q: What is CloudWatch Synthetics Canary?**
A: A feature that runs scripts (Node.js/Python with headless Chrome) to programmatically simulate customer actions against your APIs and websites, alerting you to failures before customers encounter them.

**Q: What languages can Synthetics Canary scripts be written in?**
A: **Node.js** or **Python** — with access to a headless Google Chrome browser.

**Q: How would you use Synthetics Canary for automatic failover?**
A: Canary detects failure → triggers CloudWatch Alarm → invokes Lambda → Lambda updates Route 53 DNS record to redirect traffic to a healthy region.

**Q: What blueprint would you use to detect broken links on a website?**
A: **Broken Link Checker** blueprint — checks all links within a URL and flags any that are broken.

**Q: What blueprint auto-generates a script by recording browser actions?**
A: **Canary Recorder** (used with CloudWatch Synthetics Recorder).

---

## Quick Reference

```
Synthetics Canary = scripted monitoring of APIs/URLs/web flows
Runtime: Node.js or Python + headless Chrome
Schedule: one-time or recurring

Blueprints:
  Heartbeat Monitor    → load URL, screenshot, HTTP archive
  API Canary           → test REST API read/write
  Broken Link Checker  → find broken links in a URL
  Visual Monitoring    → screenshot diff against baseline
  Canary Recorder      → record actions → auto-generate script
  GUI Workflow Builder → test multi-step webpage interactions

Failure flow: Canary fails → CloudWatch Alarm → Lambda → action (e.g., Route 53 failover)
```
