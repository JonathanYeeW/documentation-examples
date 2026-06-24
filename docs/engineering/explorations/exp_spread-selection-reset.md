# Spread Selection Reset on Bread Change

**Ticket:** ENG-312
**Date:** 2026-03-05
**Explorer:** Jamie Chen
**Status:** Complete — root cause identified, fix recommended

## 🔎 Problem

Over the first week after launching Artisan Spread Selection, we received 14 support tickets from users saying their spread choice "disappeared." The pattern across all 14 reports was the same: user selects a spread, moves forward in the order flow, navigates back to change their bread type, then moves forward again — and the spread selector shows no selection. The user has to re-pick their spread.

11 of the 14 reports involved sunflower butter. The other three were almond butter. No peanut butter users reported the issue.

The bug doesn't block orders — users can re-select their spread and continue. But it's frustrating, especially for users who don't notice the reset and end up confirming an order with no spread selected, which throws an error at assembly time.

## 🧭 Flow at a Glance

```
PHASE 1: STEP RENDERING (component lifecycle)

  1. User lands on a step → step component mounts
  2. User navigates away → step component unmounts

        ~~~~~~~~ handoff to state ~~~~~~~~

PHASE 2: STATE PERSISTENCE (shared vs local)

  3. On selection change → component writes to state
  4. On remount → component reads from state to restore selection

        ~~~~~~~~ handoff to navigation ~~~~~~~~

PHASE 3: NAVIGATION (back/forward between steps)

  5. User clicks "Back" → current step unmounts, previous step mounts
  6. User clicks "Next" → current step unmounts, next step mounts
  7. Remounted step restores selection from state
```

---

## Phase 1: Step Rendering

The order flow renders one step at a time. Each step — bread, jelly, spread, confirmation — is a separate component. When the user navigates between steps, the current step's component unmounts entirely and the next step's component mounts fresh. There's no hidden rendering or background persistence — if a component isn't on screen, it doesn't exist in the DOM.

**Step 1: Mount.** When the user arrives at a step, the corresponding component mounts and initializes. `BreadSelector`, `JellySelector`, and `SpreadSelector` each render their options and check for any existing selection to restore.

**Step 2: Unmount.** When the user navigates away, the step component unmounts. Any state held locally inside the component (via `useState`) is destroyed. This is standard React behavior — local state doesn't survive unmount.

---

## Phase 2: State Persistence

The order flow maintains a shared state object at the flow level that persists across step navigation. Individual step components are responsible for writing their selections to this shared state and reading from it on mount.

**Step 3: Write on change.** When a user makes a selection, the step component writes the value to the shared order state. `BreadSelector` writes to `orderState.breadType`. `JellySelector` writes to `orderState.jellyFlavor`. This happens on every selection change so the shared state is always current.

**Step 4: Read on mount.** When a step component mounts (or remounts after back-navigation), it checks the shared order state for an existing value. If one exists, it restores that selection in the UI. This is what makes bread and jelly selections survive navigation — the component dies, but the selection lives in shared state and gets restored when the component comes back.

---

## Phase 3: Navigation

Back and forward navigation triggers the mount/unmount cycle from Phase 1 and relies on the state persistence from Phase 2 to keep selections intact.

**Step 5: Back.** User clicks "Back" from any step. The current step component unmounts, the previous step component mounts. The previous step reads from shared state and restores its selection.

**Step 6: Forward.** User clicks "Next" from any step. Same cycle — current unmounts, next mounts.

**Step 7: Restore.** The remounted step reads its selection from shared order state and shows it as selected. The user sees their previous choice still in place.

## 🔍 Evidence

Three sources helped narrow down what was happening: support ticket patterns, client-side logs from affected sessions, and the `SpreadSelector` component source compared against the existing step components.

### Support Tickets

All 14 tickets follow the same reproduction path — user selects spread, moves forward, goes back to change bread, moves forward again, spread is gone. No tickets from users who navigated back to change jelly, which makes sense — jelly is the step immediately before spread, so going back to jelly doesn't unmount and remount the spread selector. The bug only surfaces when the user skips back far enough that the spread step fully unmounts.

The spread distribution across tickets: 11 sunflower butter, 3 almond butter, 0 peanut butter. At first glance this looks spread-specific, but order volume data shows sunflower butter at 22% of orders and almond butter at 8%. The report rates are proportional to adoption, not to a spread-specific defect.

### Client-Side Logs

Pulled navigation event logs from five affected sessions. The pattern in every session:

```
[14:23:01] SpreadSelector mounted — selectedSpread: null
[14:23:08] User selects "sunflower_butter" — local state updated
[14:23:12] User clicks "Next" — SpreadSelector unmounts
[14:23:15] User clicks "Back" — navigates to BreadSelector
[14:23:22] User changes bread to "wheat"
[14:23:25] User clicks "Next" — JellySelector mounts, reads orderState.jellyFlavor: "grape" ✓
[14:23:27] User clicks "Next" — SpreadSelector mounts — selectedSpread: null ✗
```

The jelly selection survives the round trip (restored from `orderState.jellyFlavor`). The spread selection doesn't — `SpreadSelector` mounts with `null` because there's nothing in shared state to read from. The local state from the previous mount is gone.

### Component Source Comparison

Side-by-side read of the three step components:

| Component | Writes to shared state | Reads from shared state on mount | Local useState |
|---|---|---|---|
| `BreadSelector` | `orderState.breadType` on change | Yes — restores on mount | No |
| `JellySelector` | `orderState.jellyFlavor` on change | Yes — restores on mount | No |
| `SpreadSelector` | None | None | Yes — `useState(null)` |

`SpreadSelector` is the only component managing selection in local state. The shared order state object doesn't have a `spreadType` field — it was never added.

## 📍 Findings

`SpreadSelector` doesn't follow the shared state pattern. It stores its selection in local component state (`useState`) and never writes to the shared order state. The shared state object has fields for `breadType` and `jellyFlavor` but no field for `spreadType` — it was never added when the Artisan Spread Selection feature shipped. When the user navigates away from the spread step, `SpreadSelector` unmounts, its local state is destroyed, and there's nothing in shared state to restore from when it remounts.

This affects all three spreads equally. Sunflower butter is over-represented in reports because peanut butter is the first option in the list — when `SpreadSelector` remounts with no selection, peanut butter's card gets default focus styling, which looks like it's still selected. Peanut butter users never notice the reset. Almond butter has only 3 reports because its adoption is about 8% of orders vs. sunflower butter's 22%.

The fix is adding `spreadType` to the shared order state and updating `SpreadSelector` to read and write from it on selection change, matching the pattern `BreadSelector` and `JellySelector` already use. Beyond the immediate fix, the shared state contract should be documented so the next step component added to the flow follows the same pattern.
