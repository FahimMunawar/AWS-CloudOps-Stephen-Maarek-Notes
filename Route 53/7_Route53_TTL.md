# 7 — Route 53: TTL (Time To Live)

## How TTL Works

```
Client → DNS query → Route 53 returns: IP + TTL (e.g., 300 seconds)
  → Client caches result for 300 seconds
    → During TTL: client uses cached IP, no new DNS query
    → After TTL expires: client queries Route 53 again
```

---

## TTL Trade-offs

| TTL | DNS Traffic | Record Change Lag | Use Case |
|---|---|---|---|
| **High** (e.g., 24h = 86400s) | Low (fewer queries → less cost) | Up to 24h for clients to see new record | Stable records |
| **Low** (e.g., 60s) | High (more queries → higher cost) | Max 60s before clients see new record | Frequent changes |

---

## Strategy for Changing Records Safely

```
1. Reduce TTL to low value (e.g., 60s) → wait 24h (old high TTL to expire)
2. Change the record value
   → All clients pick up new value within 60 seconds
3. Raise TTL back to normal (e.g., 3600s)
```

---

## Key Facts

- TTL is **mandatory** for all record types **except Alias records**
- Even if you update a record in Route 53, clients retain the old IP until their cached TTL expires
- `dig` output shows the **remaining TTL** — decrements with each query during the cache window

### Verifying TTL with dig

```bash
dig demo.example.com
# Answer section shows: record + TTL (e.g., 115)
# Run again seconds later → TTL decrements (e.g., 98)
# After TTL hits 0 → new query to Route 53 → fresh result
```

---

## Quick Reference

```
TTL = how long clients cache the DNS answer (in seconds)

High TTL → less Route 53 cost, slower record updates
Low TTL  → more Route 53 cost, faster record updates

Record change strategy:
  Lower TTL → wait for old TTL to expire → change record → raise TTL

TTL mandatory on all records EXCEPT Alias records
dig shows remaining TTL countdown — useful for confirming cache behavior
```
