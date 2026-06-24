# The Quality Model

**Created:** 2026-04-11
**Last Updated:** 2026-06-23

---

# Summary

Quality in the PB&J system is not a single check at the end — it's a philosophy of failing as early as possible and never serving a sandwich that doesn't meet the bar. Understanding this model is important when deciding where to add new quality checks, how to handle failures gracefully, and why certain decisions (like running freshness checks before assembly starts) were made the way they were.

---

# Model at a Glance

```
GATE 1: INGREDIENT GATE
  Runs before assembly begins.
  Checks freshness and availability.
  Failure cost: nothing wasted.

GATE 2: COVERAGE GATE
  Runs after spread and jelly are applied.
  Checks for bare spots and uneven application.
  Failure cost: bread and spread wasted.
  → Recoverable via touch-up pass; unrecoverable = order fails.

GATE 3: FINAL GATE
  Runs after cutting and plating.
  Confirms the finished sandwich meets spec.
  Failure cost: everything wasted.
  → A failure here signals a gap in earlier gates, not a routine catch.
```

---

# The Three Quality Gates

The following describes each gate in sequence — what it checks, when it runs, and why it's positioned where it is. Gates are ordered by cost of failure: the earlier a problem is caught, the cheaper it is to handle.

## Gate 1: Ingredient Gate

The ingredient gate asks: do we have what we need, and is it good? It runs before a single step of assembly begins. A failed ingredient gate means nothing was wasted except a few seconds of check time.

This is the gate that matters most for operational efficiency. Catching a stale jelly jar before assembly starts costs nothing. Catching it after the peanut butter is already on the bread wastes the bread and the prep time. Any check that can run before assembly should run here.

## Gate 2: Coverage Gate

The coverage gate asks: did the application go correctly? It runs after spread and jelly are on the bread, before the slices are combined. At this point ingredients have been committed — a failure here wastes the bread and the spread. But it's still cheaper than plating a bad sandwich and having the user see it.

Coverage failures are recoverable with a single touch-up pass. Only failures that can't be resolved after the touch-up escalate to order failure.

## Gate 3: Final Gate

The final gate asks: is this what the user ordered? It runs after cutting and plating, before the order is marked complete. This gate is a confirmation, not a safety net.

If the system is working correctly, nothing should fail here that didn't already fail at an earlier gate. When the final gate does catch something, it usually indicates a new failure mode the earlier gates weren't checking for — and that's a signal to add a check upstream, not to rely on the final gate as a permanent catch.

---

# Failure Cases

The following describes how quality failures are handled at each gate.

## Recoverable Failures

Some failures can be resolved without failing the order. Coverage gaps at Gate 2 trigger a touch-up pass and recheck. If the touch-up resolves the issue, assembly continues and the failure is never surfaced to the order state machine — from the lifecycle's perspective, it didn't happen.

## Unrecoverable Failures

A failure is unrecoverable when the pipeline has exhausted its retry or correction options. At Gate 1, an ingredient that is stale or missing cannot be substituted — the order fails before assembly begins. At Gate 2, a coverage failure that survives the touch-up pass cannot be fixed without restarting — the order fails and ingredients are wasted. At Gate 3, any failure is unrecoverable by definition — the sandwich is already plated.

In all cases, the order transitions to `FAILED`, the user is notified with a specific reason, and the failure is logged for operational review.

---

# Design Decisions

The following captures the intentional choices behind how the quality model is designed — why the gates are ordered the way they are, and what the quality system is and isn't responsible for.

## Why This Order Matters

The gates are sequenced by cost of failure. This means checks that can run before assembly always run before assembly, checks that require partially assembled state run mid-assembly, and the final gate exists to confirm rather than to catch.

When adding a new quality check, the first question is: what is the earliest point in the pipeline where this failure is detectable? That's where the check belongs. Adding a detectable-early check at a late gate will still catch failures — but it will do so expensively.

## What Quality Is Not

Quality is not about perfection. The coverage gate accepts any coverage above its threshold — it doesn't try to produce a mathematically perfect spread application. The freshness system uses date-based windows rather than continuous monitoring because the marginal quality improvement from continuous monitoring doesn't justify the complexity.

Quality is also not about the user's taste preferences. Whether someone prefers more jelly or less spread is an order configuration concern, not a quality concern. The quality system enforces a minimum bar, not an optimum.
