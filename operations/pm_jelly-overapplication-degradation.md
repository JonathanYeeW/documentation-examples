# Jelly Overapplication on Large-Format Bread Orders

**Incident ID:** INC-2301
**Date Opened:** 2026-03-03
**Date Closed:** 2026-03-09
**Severity:** P2 — Degraded Experience
**Duration:** 6 days (2026-03-03 to 2026-03-09)
**Authors:** Jordan Kim (product engineer), Casey Nguyen (product manager)
**Status:** Resolved — action items in progress

---

## 📋 Summary

For six days following a performance update to the jelly applicator, orders on large-format bread types (sourdough and artisan loaf) received approximately 2.4x the intended amount of jelly — enough to make the sandwich soggy by the time it reached the customer. The system never went down. Every order was confirmed and fulfilled. But for roughly 1 in 7 orders during this period, the sandwich that arrived was noticeably worse than it should have been.

The issue was discovered not through an alert, but through a drop in sandwich quality ratings that the product team noticed during a weekly metrics review. By the time it was caught, 847 orders had been affected.

---

## 💥 Impact

**Product quality:** 847 orders were fulfilled with excess jelly. Customer feedback from this period describes sandwiches as "soggy," "falling apart," and "wet in the middle." These are the exact complaints the applicator system was designed to prevent.

**Ratings:** Average sandwich quality score dropped from 4.6 to 3.9 over the incident window — a 0.7-point decline. On large-format bread orders specifically, the average score was 3.4. This is the lowest sustained quality rating since launch.

**Customer trust:** Unlike an outage, where customers can be refunded and contacted, a quality degradation is harder to make whole. Customers who had a bad experience may not return, and we can't easily identify every affected customer with enough certainty to make proactive outreach feel genuine rather than alarming.

**Revenue:** No direct revenue was lost — all orders were paid for and fulfilled. The downstream risk is repeat order rate. We are monitoring whether affected customers reordered in the 14 days following the incident.

---

## ⏱️ Timeline

| Date / Time | Event |
|---|---|
| Mar 3, 09:15 UTC | Performance update to the jelly applicator deployed. The update is intended to reduce application time on standard bread orders by ~400ms. Change is reviewed and approved as low-risk — it only touches timing and pressure parameters, not volume. |
| Mar 3, 09:20 UTC | First affected orders begin processing. Large-format bread orders start receiving excess jelly. No alert fires. Assembly pipeline metrics show normal throughput. |
| Mar 4–6 | Orders continue. Quality ratings begin a gradual decline. No single day shows a sharp drop sharp enough to trigger existing alerting thresholds. |
| Mar 7, 14:00 UTC | Weekly product metrics review. Casey Nguyen flags that the 7-day quality score trend is down 0.7 points. Notes it correlates loosely with the applicator deployment but doesn't yet have enough signal to be certain. |
| Mar 8, 10:00 UTC | Jordan Kim pulls order-level quality data and segments by bread type. Large-format bread orders show a 1.2-point average quality score drop; standard bread orders show no change. Pattern is clear. |
| Mar 8, 11:30 UTC | Jordan reviews the applicator deployment. Finds the pressure parameter update didn't account for the larger surface area of sourdough and artisan loaves — the same pressure that applies 1.5 tbsp to standard bread applies ~3.6 tbsp to large-format bread. |
| Mar 9, 08:00 UTC | Fix deployed. Jelly application volume is now calculated based on bread surface area, not a fixed pressure value. |
| Mar 9, 08:30 UTC | Manual quality check on 10 post-fix large-format orders confirms correct jelly volume and structural integrity. |
| Mar 9 onward | Quality scores on large-format orders begin recovering. Monitoring continues. |

---

## 🔍 Root Cause

The performance update to the jelly applicator changed how pressure was applied during dispensing but did not account for the variable surface area of different bread types.

The applicator had always used a fixed pressure value — calibrated against standard sandwich bread — to dispense approximately 1.5 tablespoons of jelly. That calibration assumed a specific bread surface area. Standard bread slices are consistent enough that this worked fine. Large-format bread (sourdough, artisan loaf) has a surface area roughly 2.4x larger. At the same pressure setting, the applicator doesn't stop at 1.5 tablespoons — it keeps dispensing until coverage is detected across the full surface, resulting in ~3.6 tablespoons on large-format slices.

This relationship between pressure and volume had always been implicit in the system — it worked on standard bread by coincidence of calibration, not by design. The performance update didn't introduce the underlying fragility; it exposed it. The fix makes the calculation explicit: jelly volume is now calculated directly from detected bread surface area, making the behavior correct and predictable for any bread type.

---

## 🔎 Detection

The incident was detected six days after it began, during a routine weekly metrics review. There was no alert.

Our existing quality alerting watches for single-day score drops below a fixed threshold. This incident produced a gradual decline spread across six days — no single day crossed the threshold. The alerting was calibrated for acute drops, not slow degradation.

The product manager caught it by looking at a 7-day trend chart rather than a daily snapshot. That was the right instinct, but it was manual, dependent on someone being in the right meeting and looking at the right chart on the right week.

---

## 🛠️ Response

Once the pattern was confirmed on March 8th, the investigation moved quickly. Segmenting quality scores by bread type isolated the issue to large-format bread within an hour. Reviewing the recent deployment history pointed immediately to the applicator update. Understanding the root cause — fixed pressure, variable surface area — took another hour of reading the applicator source code.

The fix itself was straightforward: replace the fixed pressure target with a surface-area-based volume calculation. It was built, tested, and deployed the following morning.

The harder part was scoping the impact. Because the sandwich was physically made and delivered, there was no clean log of "failed orders" to pull. The 847 figure comes from joining quality score data with order metadata — large-format bread orders during the incident window that received a quality rating below 4.0. There is some uncertainty in this number: not every affected customer left a rating, and not every low rating during this period was caused by jelly volume.

---

## ✅ What Went Well

- **Fast fix once identified.** From root cause confirmation to deployed fix was less than 24 hours. The applicator code was well-documented enough that the surface-area calculation was a clean, low-risk change.
- **Strong diagnostic signal in quality ratings.** Once Casey flagged the trend and Jordan segmented by bread type, the data pointed clearly to the problem. The quality rating system worked as a signal even without a dedicated alert.
- **No safety or health issues.** Excess jelly is a quality problem, not a food safety problem. The ceiling on customer harm here was a bad experience — not something more serious.

---

## ❌ What Went Wrong

- **The applicator assumed a fixed bread size.** The surface-area-to-volume relationship was never explicitly modeled. It happened to work for standard bread. The system was one edge case away from failing, and we didn't know it.
- **The deployment was treated as low-risk without sufficient testing on large-format orders.** The change was reviewed against standard bread behavior only. Large-format bread was not tested in staging.
- **We had no alerting for gradual quality degradation.** Our alerts catch acute drops. A slow decline over days went unnoticed until a human happened to look at the right chart.
- **Six days is a long time to be serving a degraded product without knowing it.** Hundreds of customers had a bad experience that we weren't aware of and couldn't respond to in real time.

---

## 🍀 Where We Got Lucky

- The weekly metrics review happened to fall on day five of the incident. If the review had been biweekly, this could have run for two weeks.
- The quality rating adoption is high enough that we had sufficient data to segment by bread type with statistical confidence. If rating adoption were lower, the signal might have been too noisy to isolate.
- The issue affected jelly volume, not the spread applicator. Peanut butter overapplication would have introduced an allergy risk for customers with tree nut or peanut sensitivities. Jelly is lower-stakes.

---

## 📌 Action Items

| Action | Type | Owner | Priority | Status |
|---|---|---|---|---|
| Add trend-based quality alerting — page on-call if 3-day average score drops more than 0.3 points from the prior 7-day average | Detect | Jordan Kim | P0 | In progress |
| Expand staging test suite to include large-format bread orders for all applicator changes | Prevent | Jordan Kim | P1 | Not started |
| Audit all applicator calibrations for implicit assumptions about bread size or surface area | Prevent | Jordan Kim | P1 | Not started |
| Define a postmortem trigger for quality degradation incidents — currently postmortems are only required for outages | Process | Casey Nguyen | P1 | Not started |
| Evaluate proactive outreach to affected customers — determine if we have enough confidence in the 847 figure to contact them | Respond | Casey Nguyen | P2 | In discussion |
| Add bread surface area as a first-class metadata field in the order system — applicators should always know what they're working with | Prevent | Jordan Kim | P2 | Not started |
