# feat: order service graceful degradation on spread registry unavailability

**Ticket:** ENG-315
**Type:** Reliability
**Priority:** High
**Reporter:** Platform Team
**Created:** 2026-03-06

## 👤 Service Journey

When a customer confirms an order, the Order Service calls the Spread Registry to validate the selected spread before handing off to the Assembly Pipeline. This is a hard dependency — if the Spread Registry is down or times out, the Order Service has no fallback. The order fails with a 500 and the customer sees an error on the confirmation screen.

The Spread Registry is a read-heavy, low-churn service. The full spread list changes infrequently — a new spread every few months, a disable/enable a few times a year. The Order Service is calling it on every single order confirmation, treating a near-static config as a live dependency that must be reachable at transaction time.

## 🔄 Current vs. Expected

```
Current:

  Customer confirms order
  → Order Service calls Spread Registry: GET /spreads/:id
  → Spread Registry is unavailable (deploy, timeout, crash)
  → Order Service receives no response
  → Order fails with 500
  → Customer sees error on confirmation screen

Expected:

  Customer confirms order
  → Order Service calls Spread Registry: GET /spreads/:id
  → Spread Registry is unavailable
  → Order Service falls back to local cache
  → Order proceeds normally
  → Spread Registry unavailability is logged for ops visibility
  → Customer never sees an error
```

## 🐛 Problem

The Order Service treats the Spread Registry as a required live dependency at order confirmation time. A transient registry outage — a deploy, a timeout, a crash — directly causes customer-facing order failures. Given that spread config is nearly static, there's no reason the Order Service can't serve from a local cache when the registry is temporarily unreachable.

## ✅ Acceptance Criteria

```
1. Order Service maintains a local cache of spread config
   → Cache is populated on service startup
   → Cache refreshes on a configurable interval (default: 5 minutes)

2. On Spread Registry unavailability, Order Service falls back to cache
   → Orders proceed using cached spread config
   → No customer-facing error is surfaced during a registry outage

3. Cache miss with registry unavailable fails safely
   → If a spread ID is not in cache and registry is unreachable,
     order fails with a clear error — not a silent bad state

4. Registry unavailability is observable
   → Every fallback-to-cache event is logged with spread ID,
     timestamp, and reason (timeout, 5xx, connection refused)

5. Cache does not serve stale disabled spreads indefinitely
   → If a spread is disabled in the registry, the cache reflects
     this within one refresh interval
   → A customer cannot order a disabled spread due to stale cache
     more than 5 minutes after it was disabled
```
