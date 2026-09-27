# [ENG-331] fix: jelly amount follows the bread on large loaves

**Ticket:** ENG-331 · **PR:** #52

## Context

**Where this sits in the journey**

- A customer orders a sandwich and picks their bread. Sourdough and artisan loaf are the two large-format breads.
- The machine spreads the jelly, closes the sandwich and hands it over. The customer rates it afterwards.

**Terms**

- **Large-format bread** — sourdough and artisan loaf, whose slices are about half again as wide as the others.
- **Jelly amount** — how much jelly goes on one sandwich. It's meant to scale with the slice, so every sandwich ends up equally covered.

## Summary

On sourdough and artisan loaf, customers have been getting about two and a half times the jelly they should, and their sandwiches arrive soggy. Nothing fails along the way, so the only sign is a bad sandwich and a low rating. This is the fix for INC-2301. This PR makes one change:

1. **Jelly follows the bread** — a customer who orders a large loaf gets jelly for the size of their slice, the same coverage as every other bread.

## Change 1: Jelly follows the bread

A customer who orders sourdough or artisan loaf gets a sandwich with the same jelly coverage as any other bread.

```
Current:

  A customer orders sourdough with grape jelly
  → The machine builds it, with no errors
  → Jelly for a slice more than twice the size
  → The sandwich arrives soggy, and gets rated 3 stars

New:

  A customer orders sourdough with grape jelly
  → The machine builds it, with no errors
  → Jelly for a slice this size                           ← fixed
  → The sandwich arrives like any other
```

- Every other bread gets exactly the jelly it did before.
- Nothing changes for the customer while ordering. They only see the difference in the sandwich.
- Customers who already got a soggy sandwich are handled in INC-2301's follow-up, not here.
