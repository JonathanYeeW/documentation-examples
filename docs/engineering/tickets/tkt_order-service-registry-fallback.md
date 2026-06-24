# feat: order service graceful degradation on spread registry unavailability

**Ticket:** ENG-315
**Type:** Reliability
**Priority:** High
**Created:** 2026-03-06

## 🔴 The Problem

A transient Spread Registry outage causes every in-flight order confirmation to fail with a customer-facing error, even though spread config is nearly static and could be served from a local cache.

## 📍 Experience Today

```
Customer confirms order
→ Order Service calls Spread Registry to validate spread
→ Spread Registry is unavailable (deploy, timeout, crash)
→ Order Service receives no response
→ Order fails with 500
→ Customer sees error on confirmation screen
```

## ✅ Experience After

```
Customer confirms order
→ Order Service calls Spread Registry to validate spread
→ Spread Registry is unavailable
→ Order Service falls back to local cache
→ Order proceeds normally
→ Outage is logged for ops visibility
→ Customer never sees an error
```
