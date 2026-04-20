# 6 — Route 53: Email DNS Records

## Record Types for Email

| Record | Name | Purpose |
|---|---|---|
| **MX** | Mail Exchange | Tells where to deliver **inbound** email |
| **TXT (SPF)** | Sender Policy Framework | Authorizes which servers can **send** mail on behalf of your domain |
| **TXT (DKIM)** | DomainKeys Identified Mail | Cryptographically verifies **message integrity** |
| **TXT (DMARC)** | Domain-based Message Auth | Defines handling policy for messages failing SPF or DKIM checks |

> **Exam key**: SPF and DKIM are both **TXT records**. MX is for **receiving** email.

---

## Amazon SES + Route 53 Integration Flow

1. Verify your domain in Amazon SES → get a **TXT record** to prove domain ownership
2. Set up DKIM → get **3 CNAME records** for DKIM signature
3. Add all records to the Route 53 hosted zone:
   - SES verification TXT record
   - DKIM CNAME records (×3)
   - SPF TXT record (authorizes SES to send)
   - MX record (if receiving email via SES)
4. Wait for DNS propagation → SES status changes to **Verified**

> SES provides a one-click option to create all Route 53 records automatically.

---

## Quick Reference

```
Email DNS records:
  MX            → inbound email delivery
  TXT (SPF)     → authorizes outbound senders (e.g., include:amazonses.com)
  TXT (DKIM)    → cryptographic message integrity verification
  TXT (DMARC)   → policy for SPF/DKIM failures

Exam:
  SPF = TXT record
  DKIM = TXT record (SES uses CNAME records for DKIM)
  MX = receiving email
  DMARC = handling policy for failed auth

SES setup: verify domain (TXT) + DKIM (3 CNAMEs) + SPF (TXT) + MX → propagate → verified
```
