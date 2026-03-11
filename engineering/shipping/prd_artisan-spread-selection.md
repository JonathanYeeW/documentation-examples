# feat: add artisan spread selection to sandwich ordering flow

**Ticket:** ENG-312  
**PR:** #47  
**Reviewer:** Sam Torres

## Summary

Adds `SpreadSelector` as the third step in the sandwich ordering flow, between `JellySelector` and the confirmation screen. Previously the system hardcoded peanut butter with no option to change it — this was the highest-voted item in the Q1 feedback survey. Three spreads are now supported: `peanut_butter`, `almond_butter`, and `sunflower_butter`. Selection persists across back-navigation using the same shared state pattern as `BreadSelector` and `JellySelector`. The ordering API now accepts `spreadType` as a required field.

## Current vs. New Behavior

```
Current:

  User opens PB&J Builder
  → Selects bread type
  → Selects jelly flavor
  → Confirms order
  → System assembles with peanut butter (hardcoded, no choice offered)

New:

  User opens PB&J Builder
  → Selects bread type
  → Selects jelly flavor
  → Selects spread from available options        ← new step
  → Confirms order
  → System assembles with chosen spread
```

## How to Test

```bash
# Start the simulator
npm run simulator

# Run the order wizard
npm run dev order
```

Walk through the full ordering flow. After selecting jelly, the spread step should appear with two visible options (`peanut_butter` and `almond_butter`). Select one and continue to confirmation — the selection should appear in the summary. Navigate back to the jelly step and forward again — the spread selection should still be set.

To verify the API validation:

```bash
# Submit an order without a spread — should return 400
curl -X POST http://localhost:3000/orders \
  -H "Content-Type: application/json" \
  -d '{ "breadType": "sourdough", "jellyFlavor": "grape" }'

# Expected: 400 { "error": "spreadType is required" }
```

```bash
# Run the full test suite
npm test
```

New tests are in `SpreadSelector.test.tsx` and `order.handler.test.ts`. All existing tests should pass unchanged.
