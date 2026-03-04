# Order Loss During Database Migration

**Incident ID:** INC-2287
**Date:** 2026-02-14
**Severity:** P1 — Critical
**Duration:** 47 minutes (14:03–14:50 UTC)
**Authors:** Alex Rivera (on-call engineer), Sam Park (engineering manager)
**Status:** Resolved — action items in progress

---

## 📋 Summary

During a scheduled database migration on February 14th, the order service lost the ability to write new orders to the database for 47 minutes. During this window, 218 orders were accepted by the app and payment was collected — but the orders were never sent to the assembly pipeline. Customers received a confirmation screen but no sandwich, and no notification that anything had gone wrong. The issue was resolved when the on-call engineer identified the failed migration and rolled back to the previous database configuration.

---

## 💥 Impact

**Customers:** 218 orders were confirmed but never fulfilled. Customers paid for a sandwich and received nothing. 34 customers contacted support during the incident window. An additional 89 support tickets arrived in the 2 hours after resolution, as customers realized their order wasn't coming.

**Revenue:** $1,526 in payments were collected for undelivered orders. All 218 customers have been issued full refunds.

**Support:** The incident generated 123 total support tickets — roughly 8x our normal hourly volume. Support team spent approximately 6 person-hours resolving tickets that afternoon.

**Trust:** Valentine's Day was the highest-order day in company history. A meaningful number of customers were placing their first-ever order. This was a bad day for the issue to occur.

---

## ⏱️ Timeline

| Time (UTC) | Event |
|---|---|
| 13:45 | Scheduled database migration begins. Migration is expected to take 10 minutes and is considered low-risk. |
| 13:58 | Migration completes. Engineers mark it as successful and close the deployment ticket. |
| 14:03 | Order write failures begin. The order service is connecting to the old database endpoint, which no longer accepts writes. Customers see a confirmation screen regardless — a known gap in our error handling. |
| 14:31 | First support ticket arrives. Customer reports receiving a confirmation but no sandwich after 25 minutes. |
| 14:38 | Support team flags the ticket pattern to the on-call engineer — multiple customers, same complaint. |
| 14:41 | On-call engineer begins investigation. Checks order service logs and identifies a spike in database write errors starting at 14:03. |
| 14:47 | Root cause confirmed: order service is still pointing to the old database endpoint. Migration did not update the service configuration. |
| 14:50 | Configuration updated. Order service reconnects to the new database. Writes resume. |
| 15:10 | All 218 affected orders identified. Refund process initiated. |
| 15:45 | Refunds issued. Support team begins direct outreach to affected customers. |

---

## 🔍 Root Cause

The database migration moved our order data to a new database instance, but it did not update the order service's configuration to point to the new instance. The order service kept writing to the old endpoint — which was now read-only — and those writes failed silently.

Two things made this worse than it had to be:

**1. Silent failures.** When a write to the database fails, the order service currently returns a success response to the customer anyway. This is a known design gap that was deprioritized. It meant customers got a confirmation screen for an order that didn't exist, with no indication anything was wrong.

**2. No write-failure alerting.** We have alerting for database connection errors, but not for sustained write failures. The on-call engineer wasn't paged — the issue was caught only when support escalated a pattern of customer complaints, 35 minutes into the incident.

---

## 🔎 Detection

The incident was detected through support ticket volume, not automated alerting. A support agent noticed multiple tickets with identical complaints and escalated to the on-call engineer at 14:38 — 35 minutes after the failures started. Detection time was longer than it should have been because we had no alert on database write failure rate.

---

## 🛠️ Response

Once escalated, the on-call engineer identified the issue in 9 minutes by reviewing order service error logs. The fix — updating the service configuration to point to the new database — took 3 minutes to implement and deploy. Resolution was fast once the right person was looking at the right logs.

After the service was restored, the team manually queried the database to identify all orders placed during the failure window. Refunds were processed in bulk and customers were contacted directly by support.

---

## ✅ What Went Well

- **Fast resolution once escalated.** From the moment the on-call engineer started investigating to service restoration was 9 minutes. The logs were clear and the fix was straightforward.
- **Clean blast radius identification.** We were able to pull the complete list of affected orders precisely, which made the refund and outreach process clean and accountable.
- **Support team communication.** The support team handled a high volume of tickets quickly and escalated the pattern before we had any monitoring signal. Their instinct caught the incident.

---

## ❌ What Went Wrong

- **The migration didn't update the service configuration.** This was the root cause. Our migration process doesn't have a step that verifies all dependent services are pointing to the new endpoint before the old one becomes read-only.
- **No alerting on write failures.** We had 35 minutes of silent data loss before anyone knew. That's too long.
- **Order confirmation sent on write failure.** Customers should never see a success screen when their order didn't go through. This gap turned a technical failure into a trust failure.
- **Incident happened on our highest-traffic day.** The migration was scheduled without checking the calendar. Valentine's Day was flagged in the marketing calendar but not in the infrastructure release schedule.

---

## 🍀 Where We Got Lucky

- The failure window was 47 minutes, not hours. If the support team hadn't escalated, it's possible this ran through the evening.
- All 218 affected orders were recoverable from logs. We didn't lose the data — we just lost the writes. The refund and outreach process was clean because the evidence was complete.
- No orders went to the assembly pipeline in a broken state. The failure mode was "order not created" rather than "order created incorrectly." Sandwiches that were made were made correctly.

---

## 📌 Action Items

| Action | Type | Owner | Priority | Status |
|---|---|---|---|---|
| Add write failure rate alert to order service — page on-call if error rate exceeds 1% for 2 consecutive minutes | Detect | Alex Rivera | P0 | In progress |
| Fix order service to return an error response (not success) when a database write fails | Fix | Jamie Chen | P0 | In progress |
| Add a migration checklist step: verify all dependent services point to new endpoint before making old endpoint read-only | Prevent | Sam Park | P1 | Not started |
| Block infrastructure deployments on high-traffic dates — cross-reference marketing calendar during release planning | Prevent | Sam Park | P1 | Not started |
| Implement end-to-end order confirmation — only confirm to customer after order is written and assembly is queued | Fix | Jamie Chen | P2 | Not started |
| Conduct a review of other services for the same silent-failure pattern | Prevent | Alex Rivera | P2 | Not started |
