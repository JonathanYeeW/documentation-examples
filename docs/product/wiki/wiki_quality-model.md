# The Quality Model

**Created:** 2026-04-11

# 📋 Summary

Every sandwich passes three quality gates before it's served, each checking a different thing at a different point in the build. This article covers what each gate checks, what a failure there costs, and what happens next. How an order's state changes when a gate fails is covered elsewhere.

# 🧭 Context

**What the PB&J Machine is for**

Kids go hungry because the systems that feed them are slow, expensive and fragile. PB&J Machine Co. builds the software for a machine that can make any peanut butter and jelly sandwich on demand: a parent, a cafeteria worker or a food bank volunteer picks a bread, a spread and a jelly, and the machine makes it. The goal is a sandwich for any kid who needs one, with no training and no waste.

**Where this sits in the journey**

- A customer taps **Make My Sandwich**, and the machine starts building.
- While it builds, the customer watches the **progress screen**.
- Most of the time the gates pass and the customer never knows they ran. When one fails for good, the customer sees why.

**Terms**

- **Make My Sandwich** — the button at the end of the order flow that sends the order to the machine.
- **Progress screen** — the screen a customer watches after ordering, until the sandwich is ready.
- **Touch-up pass** — a second application of spread or jelly over a bare spot.

**Not covered here**

- How an order's state changes when a gate fails. That's in the order lifecycle.
- A customer's taste, like more jelly or less spread. That's part of the order they build, not quality.

# ⚙️ Section 1: How the Gates Work

A **gate** is a check that can stop a sandwich before it's served. This section is the model all three gates in Section 2 share.

**Why there are three**

- **Each gate runs at the earliest point its problem can be seen.** The earlier a problem is caught, the less is wasted.
- **A new check goes at the earliest gate that can see its problem.** A check placed later still catches the problem, but wastes more when it does.
- **A gate enforces a minimum, not perfection.** Coverage above the threshold passes. Freshness is judged by date windows rather than constant monitoring, because constant monitoring costs more than the quality it adds.

**What happens on a failure**

- **A recoverable failure is fixed, and the build carries on.** The order never sees it.
- **An unrecoverable failure fails the order.** The customer sees the specific reason, and the failure is logged for review.

**An example: a bare spot**

```
Ingredient gate    → jelly in date, bread in stock          → passes
Coverage gate      → bare spot on the left slice            → touch-up pass, rechecked, passes
Final gate         → cut, plated, matches the order         → passes, sandwich served
```

A sandwich meets the gates in build order, and Section 2 follows them in that order.

# 🚦 Section 2: The Gates

Each gate is here with what it checks and what a failure there costs.

## 2.1 Ingredient gate

"Do we have what we need, and is it good?"

- **Runs:** before assembly starts.
- **Checks:** that every ingredient is in stock and in date.
- **A failure costs:** nothing. No ingredient has been used yet.
- **On failure:** the order fails, since a missing or stale ingredient can't be swapped.
- Any check that can run before assembly runs here.

## 2.2 Coverage gate

"Did the spread and jelly go on properly?"

- **Runs:** after spread and jelly are applied, before the slices are put together.
- **Checks:** for bare spots and uneven application.
- **A failure costs:** the bread and the spread.
- **On failure:** one touch-up pass and a recheck. Only a failure that survives it fails the order.

## 2.3 Final gate

"Is this what the customer ordered?"

- **Runs:** after cutting and plating, before the order is complete.
- **Checks:** that the finished sandwich matches the order.
- **A failure costs:** everything.
- **On failure:** the order fails. A failure here means the earlier gates are missing a check, and a new check is added upstream rather than relying on this one.
