# 6. ECS — X-Ray Integration

## Overview

To use AWS X-Ray with ECS, you must run the **X-Ray Daemon** somewhere that your application containers can reach it over **UDP port 2000**. There are three patterns depending on your launch type and architecture.

---

## Pattern 1: X-Ray Daemon Container (EC2 Launch Type)

Run the X-Ray Daemon as a **Daemon task** — one container per EC2 instance.

```
EC2 Instance 1:  [App Container A]  [App Container B]  [X-Ray Daemon Container]
EC2 Instance 2:  [App Container C]  [App Container D]  [X-Ray Daemon Container]
EC2 Instance 3:  [App Container E]  [App Container F]  [X-Ray Daemon Container]
```

| Property | Detail |
|----------|--------|
| **Deployment** | One X-Ray Daemon container per EC2 instance |
| **Pattern** | ECS Daemon service — runs on every instance automatically |
| **App containers** | All app containers on the same EC2 instance share one Daemon |
| **Launch type** | EC2 only |

---

## Pattern 2: X-Ray Sidecar (EC2 Launch Type)

Run one X-Ray Daemon container **alongside each application container** in the same task definition.

```
EC2 Instance:
  Task A: [App Container] + [X-Ray Sidecar]
  Task B: [App Container] + [X-Ray Sidecar]
  Task C: [App Container] + [X-Ray Sidecar]
```

| Property | Detail |
|----------|--------|
| **Deployment** | One X-Ray Daemon per app container (in the same task) |
| **Pattern** | Sidecar — X-Ray Daemon runs side-by-side with the app |
| **Launch type** | EC2 |

---

## Pattern 3: X-Ray Sidecar (Fargate Launch Type)

With Fargate, you have no control over the underlying EC2 instances, so the **Daemon Container pattern is not available**. You must use the Sidecar pattern.

```
Fargate Task: [App Container] + [X-Ray Sidecar]
Fargate Task: [App Container] + [X-Ray Sidecar]
```

| Property | Detail |
|----------|--------|
| **Deployment** | Sidecar only — one X-Ray Daemon per Fargate task |
| **Why** | No access to underlying host — cannot deploy a host-level Daemon |
| **Launch type** | Fargate |

---

## Pattern Comparison

| | Daemon Container | Sidecar (EC2) | Sidecar (Fargate) |
|-|-----------------|---------------|-------------------|
| **X-Ray containers per instance** | 1 | 1 per app container | 1 per task |
| **Launch type** | EC2 only | EC2 | Fargate |
| **Resource efficiency** | Higher — shared Daemon | Lower — one per container | Required for Fargate |

---

## Task Definition — Sidecar Configuration

Three things required to wire up X-Ray in a task definition:

### 1. X-Ray Daemon Container — Port Mapping

```json
{
  "name": "xray-daemon",
  "image": "amazon/aws-xray-daemon",
  "portMappings": [
    {
      "containerPort": 2000,
      "protocol": "udp"
    }
  ]
}
```

> **Port 2000 / UDP** — the X-Ray Daemon always listens on this port.

### 2. App Container — Environment Variable

```json
{
  "name": "scorekeep-api",
  "environment": [
    {
      "name": "AWS_XRAY_DAEMON_ADDRESS",
      "value": "xray-daemon:2000"
    }
  ],
  "links": ["xray-daemon"]
}
```

> `AWS_XRAY_DAEMON_ADDRESS` tells the X-Ray SDK where to find the Daemon. The value resolves the `xray-daemon` hostname via the container link.

### 3. Container Link

The `links` field connects the two containers so the app container can resolve the hostname `xray-daemon`.

---

## Three Key Requirements (Exam Focus)

| Requirement | Detail |
|------------|--------|
| **Port mapping** | X-Ray Daemon container port = **2000**, protocol = **UDP** |
| **Environment variable** | App container sets `AWS_XRAY_DAEMON_ADDRESS` = `xray-daemon:2000` |
| **Container link** | App container links to `xray-daemon` to resolve the hostname |

---

## Best Practices

✓ **Use Daemon Container pattern on EC2** — more resource-efficient than a sidecar per container  
✓ **Use Sidecar on Fargate** — only option; Daemon pattern requires host-level access  
✓ **Always use port 2000 UDP** — this is the fixed port for the X-Ray Daemon  
✓ **Set AWS_XRAY_DAEMON_ADDRESS** — without this, the X-Ray SDK cannot locate the Daemon  

---

## SysOps Exam Focus

**Q1: "You are running ECS on Fargate and want to integrate X-Ray. Which pattern must you use?"**
- A) X-Ray Daemon Container — deploy one Daemon per EC2 instance
- B) X-Ray Sidecar — run an X-Ray Daemon container alongside each app container in the task definition
- C) X-Ray Agent — install directly on the Fargate host
- D) No integration needed — Fargate automatically sends traces to X-Ray
- **Answer: B** — Fargate has no host access, so you cannot use the Daemon Container pattern; sidecar is the only option

**Q2: "What port and protocol does the X-Ray Daemon container use?"**
- A) Port 443 / TCP
- B) Port 8080 / HTTP
- C) Port 2000 / UDP
- D) Port 9000 / TCP
- **Answer: C** — X-Ray Daemon always listens on port 2000 UDP; this must be set in the container's port mapping

**Q3: "How does an ECS application container know where to send trace data to the X-Ray Daemon sidecar?"**
- A) It automatically discovers the Daemon via the ECS service registry
- B) You set the environment variable AWS_XRAY_DAEMON_ADDRESS pointing to the Daemon container on port 2000
- C) You configure the Daemon's IP in the task definition CPU settings
- D) The ALB routes trace data to the X-Ray Daemon
- **Answer: B** — AWS_XRAY_DAEMON_ADDRESS is the required environment variable; the hostname resolves via the container link

**Q4: "You have an ECS cluster with 10 EC2 instances, each running many application containers. You want to run X-Ray with minimum overhead. Which pattern should you use?"**
- A) X-Ray Sidecar — one Daemon per app container
- B) X-Ray Daemon Container — one Daemon per EC2 instance (ECS Daemon service)
- C) Install the X-Ray Agent directly on each EC2 instance outside of ECS
- D) Use Fargate so you don't need to manage the Daemon
- **Answer: B** — The Daemon Container pattern deploys one X-Ray container per EC2 instance, shared by all app containers on that instance — more efficient than one sidecar per container

---

## Quick Reference

```
ECS + X-Ray integration patterns:

Pattern 1: Daemon Container (EC2 only)
  → One X-Ray Daemon container per EC2 instance
  → All app containers on that instance share it
  → More resource-efficient

Pattern 2: Sidecar (EC2)
  → One X-Ray Daemon container per app container (same task)
  → Less efficient but supported

Pattern 3: Sidecar (Fargate — required)
  → Only option for Fargate (no host access)
  → One X-Ray Daemon per Fargate task

Task definition requirements:
  1. X-Ray Daemon port mapping: 2000 / UDP
  2. App container env var: AWS_XRAY_DAEMON_ADDRESS = xray-daemon:2000
  3. App container links: ["xray-daemon"]

Exam tip:
  Fargate → sidecar only
  EC2 → Daemon Container (preferred, efficient) or Sidecar
  Port 2000 UDP + AWS_XRAY_DAEMON_ADDRESS = key config details
```

---

**File: 6_ECS_X-Ray_Integration.md**
**Status: SysOps-focused, exam-ready, concise format**
