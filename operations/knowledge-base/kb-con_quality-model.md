# The Quality Model

**Created:** 2026-04-11
**Last Updated:** 2026-04-11

Quality in the PB&J system is not a single check at the end — it's a philosophy of failing as early as possible and never serving a sandwich that doesn't meet the bar. Understanding this model is important when deciding where to add new quality checks, how to handle failures gracefully, and why certain decisions (like running freshness checks before assembly starts) were made the way they were.

## What Quality Means Here

A quality sandwich is one where every ingredient is fresh, every application is even, and the finished product looks like what the user expected. The system doesn't try to recover from a bad sandwich — it tries to prevent one from being made in the first place.

The core principle: **fail early and fail completely.** A partial sandwich that almost makes it is worse than an order that never started, because a partial failure wastes ingredients and assembly time without producing anything the user can eat. Every quality gate in the system is positioned as early as possible in the process.

## The Three Quality Gates

```
1. Ingredient Gate → before assembly begins
2. Coverage Gate   → after spread and jelly are applied
3. Final Gate      → after the sandwich is cut and plated
```

These gates are not redundant — each one catches a different class of problem that the previous gate cannot see.

**The Ingredient Gate** asks: do we have what we need, and is it good? It runs before a single step of assembly begins. A failed ingredient gate means nothing was wasted except a few seconds of check time. This is the gate that matters most for operational efficiency — catching a stale jelly jar before assembly starts costs nothing. Catching it after the peanut butter is already on the bread wastes the bread and the prep time.

**The Coverage Gate** asks: did the application go correctly? It runs after spread and jelly are on the bread, before the slices are combined. At this point ingredients have been committed — a failure here wastes the bread and the spread. But it's still cheaper than plating a bad sandwich and having the user see it. Coverage failures are recoverable with a touch-up pass; only unrecoverable failures escalate to order failure.

**The Final Gate** asks: is this what the user ordered? It runs after cutting and plating, before the order is marked complete. This gate is a confirmation, not a safety net. If the system is working correctly, nothing should fail here that didn't already fail at an earlier gate. When the final gate does catch something, it usually indicates a new failure mode the earlier gates weren't checking for — and that's a signal to add a check upstream.

## Why This Order Matters

The gates are sequenced by cost of failure. The earlier a failure is caught, the cheaper it is to handle. This means:

- Checks that can run before assembly always run before assembly
- Checks that require partially assembled state run mid-assembly
- The final gate exists to confirm, not to catch

When adding a new quality check, the first question is: what is the earliest point in the pipeline where this failure is detectable? That's where the check belongs. Adding a detectable-early check at a late gate is a missed optimization — it will still catch failures, but it will do so expensively.

## What Quality Is Not

Quality is not about perfection. The coverage gate accepts any coverage above its threshold — it doesn't try to produce a mathematically perfect spread application. The freshness system uses date-based windows rather than continuous monitoring because the marginal quality improvement from continuous monitoring doesn't justify the complexity.

Quality is also not about the user's taste preferences. Whether someone prefers more jelly or less spread is an order configuration concern, not a quality concern. The quality system enforces a minimum bar, not an optimum.
