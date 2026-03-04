# Peanut Butter Supplier API Outage

**Incident ID:** INC-2314
**Date:** 2026-03-18
**Severity:** P1 — Partial Outage
**Duration:** 2 hours 11 minutes (10:22–12:33 UTC)
**Authors:** Riley Okafor (on-call engineer), Dana Moss (engineering manager)
**Status:** Resolved — action items in progress

---

## Summary

On March 18th, NutCo — the third-party supplier that provides our peanut butter inventory API — experienced an outage on their end lasting 2 hours and 11 minutes. During this window, our system could not confirm peanut butter availability for incoming orders. Rather than degrade gracefully, the order service failed hard: every order that included peanut butter was rejected at checkout with a generic error message. 1,203 orders were dropped. Orders that did not include peanut butter — jelly-only, almond butter, sunflower butter — were unaffected and processed normally.

The underlying cause of the NutCo outage is theirs to own. What is ours to own is that a single supplier API going down took out more than a third of our order volume, with no fallback, no customer communication, and no way to queue orders for later fulfillment.

---

## Impact

**Customers:** 1,203 orders were rejected at checkout. Customers saw a generic "Something went wrong" error with no explanation. No orders were partially processed — payment was never collected for affected orders. Customers could retry successfully once the outage resolved, but had no way to know when that would be.

**Order volume:** Peanut butter is included in 38% of all orders. During a normal Tuesday morning window of this length, we would expect approximately 1,600 orders. We fulfilled 397. The outage effectively removed our most popular product from the menu for over two hours without any in-app communication to customers.

**Revenue:** Approximately $8,400 in orders were dropped based on average order value. This is potential revenue, not collected-and-refunded revenue — payment was never taken. However, some portion of those customers ordered from a competitor during the window or did not return.

**Support:** 74 support tickets were filed during the incident window — customers confused by the checkout error. Support volume was manageable because the failure was at checkout (customers knew immediately something was wrong) rather than post-confirmation.

---

## Timeline

| Time (UTC) | Event |
|---|---|
| 10:22 | NutCo API goes down. Their status page shows no immediate update. |
| 10:22 | Our inventory service begins receiving timeout errors on all peanut butter availability checks. |
| 10:23 | Alert fires: inventory service error rate exceeds 5% threshold. On-call engineer Riley Okafor is paged. |
| 10:27 | Riley confirms the error is isolated to peanut butter inventory checks. Almond butter and sunflower butter supplier APIs are healthy. Checks NutCo status page — no incident posted yet. |
| 10:31 | Riley attempts to reach NutCo on-call line. No answer. Files a support ticket through NutCo vendor portal. |
| 10:45 | NutCo posts to their status page: "Investigating API connectivity issues." No ETA given. |
| 10:50 | Dana Moss joins the incident channel. Team discusses options: manual override, disabling peanut butter orders proactively, or waiting. Decides to remove peanut butter from the active menu to stop checkout errors and set customer expectations correctly. |
| 10:58 | Peanut butter temporarily removed from the active menu. Checkout errors stop. Remaining order volume processes normally. |
| 11:15 | NutCo status page updates: "Issue identified, fix in progress." |
| 12:28 | NutCo status page: "API restored." Riley confirms inventory checks are succeeding. |
| 12:33 | Peanut butter re-added to the active menu. Normal order volume resumes. |
| 12:45 | In-app banner removed. Incident marked resolved. |

---

## Root Cause

NutCo's API outage is the triggering event, but the extent of the damage is ours to explain.

Our inventory service checks supplier APIs in real time on every order — there is no local cache, no fallback inventory value, and no graceful degradation path. When the NutCo API stopped responding, the inventory check timed out, and the order service treated a timeout the same as a confirmed "out of stock": it rejected the order. This is the wrong behavior. A supplier API being unreachable is not the same as peanut butter being unavailable. We had peanut butter in our physical inventory. We just could not confirm it programmatically.

Three compounding gaps made the impact larger than it needed to be:

**1. No cache or fallback for supplier availability.** A known quantity of peanut butter in inventory should be orderable even if the supplier API is temporarily unreachable.

**2. Timeout treated as unavailability.** The system conflated "cannot reach supplier" with "item unavailable." These are different states that should produce different behavior.

**3. No in-app communication for partial outages.** When we removed peanut butter from the menu, we had no way to tell customers why or when it would be back. The app silently stopped showing it.

---

## Detection

Detection was fast — 1 minute. The inventory service error rate alert fired at 10:23, one minute after NutCo went down. The on-call engineer was paged immediately and had confirmed the scope of the failure within 5 minutes.

Fast detection did not translate to fast mitigation, however, because we had no good options ready. The 28-minute gap between detection and taking action was spent searching for a mitigation path that did not exist. There was no runbook for supplier outages.

---

## Response

Once the scope was clear, the team weighed two options: wait for NutCo to recover while absorbing checkout errors, or manually remove peanut butter from the active menu. The team chose removal at 10:50 — 28 minutes into the incident. In hindsight this was the right call, but it took longer than it should have because there was no documented playbook and the decision required a manager to join before the team felt confident acting.

Once peanut butter was removed from the menu, the incident was largely a waiting game. Riley monitored NutCo's status page and verified API restoration before re-enabling orders at 12:33.

Post-incident, 74 support tickets were triaged and closed. No refunds were required since no payments had been collected.

---

## What Went Well

- **Alerting fired in 1 minute.** The error rate threshold caught this immediately, before the first support ticket arrived.
- **Blast radius was naturally contained.** Because supplier APIs are separate per ingredient, the NutCo outage only affected peanut butter. Almond butter, sunflower butter, and jelly orders processed normally throughout. This was not a deliberate resilience design — it is how the system happens to be structured — but it limited the damage.
- **Removing peanut butter from the menu was the right call.** Checkout errors stopped immediately and the support ticket rate dropped. Customers who arrived after 10:58 saw a reduced menu rather than a broken checkout.

---

## What Went Wrong

- **No cache or fallback for supplier availability.** This is the core architectural gap. Real-time supplier API calls with no fallback mean any supplier outage becomes our outage.
- **Timeout equaled unavailability.** The inventory service does not distinguish between "supplier says unavailable" and "supplier unreachable." It should.
- **No supplier outage runbook.** The team spent 28 minutes deciding what to do because no one had documented the response for this scenario.
- **No customer-facing communication for partial outages.** We had no mechanism to explain to customers why peanut butter disappeared or when it would return.
- **No SLA or escalation path with NutCo.** We had no contractual response time for API outages and no on-call contact that answered. Our only visibility was their public status page.

---

## Where We Got Lucky

- Payment was never collected for rejected orders. If the failure had occurred post-confirmation — as in INC-2287 — we would have been issuing refunds to over a thousand customers.
- NutCo resolved the issue in just over 2 hours. An all-day outage would have required broader customer communication and produced significantly larger revenue impact.
- The incident happened on a Tuesday morning with full staffing. The incident channel filled quickly and no decisions were delayed by coverage gaps.

---

## Action Items

| Action | Type | Owner | Priority | Status |
|---|---|---|---|---|
| Implement a short-lived local cache for supplier availability — serve cached inventory state when supplier API is unreachable rather than failing the order | Prevent | Riley Okafor | P0 | In progress |
| Distinguish timeout from unavailability in the inventory service — API unreachable should trigger fallback behavior, not order rejection | Prevent | Riley Okafor | P0 | In progress |
| Write a supplier outage runbook — document the decision tree and available actions so on-call can respond without waiting for management | Process | Dana Moss | P1 | Not started |
| Build a customer-facing partial outage banner — in-app message when a specific ingredient is temporarily unavailable | Respond | Riley Okafor | P1 | Not started |
| Renegotiate NutCo vendor contract to include API uptime SLA and an on-call escalation contact | Prevent | Dana Moss | P1 | In progress |
| Audit all third-party supplier integrations for the same timeout-as-unavailability pattern | Prevent | Riley Okafor | P2 | Not started |
| Evaluate whether a small buffer stock of high-volume ingredients could cover short supplier outages | Prevent | Casey Nguyen | P2 | Not started |
