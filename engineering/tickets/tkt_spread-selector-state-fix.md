# bug: fix spread selector to persist selection across navigation

**Ticket:** ENG-313
**Type:** Bug
**Priority:** Medium
**Reporter:** Jamie Chen
**Created:** 2026-03-06

## 👤 User Journey

A customer is building their sandwich order. They move through the flow step by step — bread, jelly, spread — then reach the confirmation screen. They decide they want a different bread type and hit back. They change their bread, move forward through jelly, and arrive back at the spread step. Their spread selection is gone. They have to pick again.

Most users catch it and re-select. Some don't — they confirm without a spread, which throws an error at assembly time and blocks their order.

## 🔄 Current vs. Expected

```
Current:

  User selects "sunflower_butter"
  → navigates forward to Confirmation
  → navigates back to change bread
  → navigates forward again
  → SpreadSelector remounts with no selection ✗

Expected:

  User selects "sunflower_butter"
  → navigates forward to Confirmation
  → navigates back to change bread
  → navigates forward again
  → SpreadSelector remounts showing "sunflower_butter" ✓
```

## 🐛 Problem

`SpreadSelector` stores its selection in local component state. When the user navigates away, the component unmounts and that state is destroyed. There's nothing to restore from on remount because `SpreadSelector` never writes to shared order state — the shared state object has no `spreadType` field.

`BreadSelector` and `JellySelector` both follow the correct pattern: write to shared state on change, read from it on mount. `SpreadSelector` was the only new component added by Artisan Spread Selection and it didn't follow the same contract.

## ✅ Acceptance Criteria

```
1. Spread selection survives back-navigation
   → User selects spread → navigates forward → navigates back past
     the spread step → navigates forward again → selection is intact

2. SpreadSelector reads from orderState.spreadType on mount
   → Existing selection is shown in the UI on remount

3. SpreadSelector writes to orderState.spreadType on selection change
   → Shared state is updated immediately on every user interaction

4. All three spreads behave consistently
   → peanut_butter, almond_butter, and sunflower_butter all persist correctly

5. No regressions to BreadSelector or JellySelector
```
