# Order Flow Navigation

**Created:** 2026-04-11
**Last Updated:** 2026-04-11
**Directory:** `pbj-app/order-flow`

The order flow is a linear wizard — bread, spread, jelly, confirmation — where each step is a separate component that mounts and unmounts as the user navigates. Navigation state, step history, and selections all live in a shared context that persists across the full session. Understanding how navigation works here is essential before modifying any step component, the back button behavior, or the confirmation screen.

Read this when you're adding a new step to the flow, debugging navigation state bugs, or modifying how the confirmation screen validates completeness.

## Part 1: 🧭 Navigation State

The order flow tracks where the user is and where they've been. This is separate from order state (what they've selected) — navigation state is about position in the flow, not selections.

**Step 1: Initialize the flow**
`OrderFlowProvider.tsx`

When the user taps to start, `OrderFlowProvider` initializes with `currentStep: 0` and `visitedSteps: []`. The step index maps to a fixed array of step components defined in `FLOW_STEPS`. Adding a new step means adding it to `FLOW_STEPS` at the correct index — the navigation logic derives everything from that array, so no other changes are required unless the new step has conditional logic.

**Step 2: Advance to the next step**
`useOrderNavigation.ts`

`goToNextStep()` increments `currentStep`, appends the previous step index to `visitedSteps`, and scrolls to the top of the screen. It does not validate that a selection was made — validation lives on the step component itself and gates the "Next" button. The navigation hook is dumb; the step is responsible for its own readiness.

**Step 3: Navigate back**
`useOrderNavigation.ts`

`goToPreviousStep()` decrements `currentStep` and pops the last entry from `visitedSteps`. It does not clear selections — going back never resets what the user has already chosen. The step component reads from shared order state on mount and restores the previous selection. If your step clears selection on back-navigation, the bug is in the component, not here.

- **At step 0 (bread)**: Back button is hidden — there's nothing to go back to
- **At any other step**: Back button is visible and enabled

## Part 2: 📋 Step Lifecycle

Each step component follows the same lifecycle: mount, restore, interact, persist, advance. This pattern is enforced by convention, not by the framework.

**Step 1: Mount and restore state**
`[StepComponent].tsx`

Every step reads its relevant field from `orderState` on mount and restores the UI to match. If `orderState.spreadType` is `"almond_butter"`, the spread step renders with the almond butter card selected. If the field is null, nothing is selected and the user must choose before advancing. This read-on-mount behavior is what makes back-navigation work correctly — the component never holds its own copy of the selection.

**Step 2: Write to shared state on change**
`[StepComponent].tsx`

When the user makes a selection, the component writes immediately to `orderState` — not only when they tap "Next." Writing on every change ensures the selection survives if the user navigates away mid-step without tapping "Next." If you move the write to happen at step completion only, back-navigation will lose the selection.

**Step 3: Gate the Next button**
`[StepComponent].tsx`

The "Next" button is disabled until the step's required field in `orderState` is non-null. The step component owns this validation — it reads directly from `orderState` and enables the button when its field is set. The navigation hook doesn't know or care whether a step is complete.

## Part 3: ✅ Confirmation and Submission

The confirmation screen is the last step. It reads the full `orderState` and validates that all required fields are present before allowing submission.

**Step 1: Validate completeness**
`ConfirmationStep.tsx`

On mount, the confirmation screen checks that `breadType`, `spreadType`, and `jellyFlavor` are all non-null in `orderState`. If any are missing, it routes the user back to the earliest incomplete step. This is a safety net — under normal navigation it should never fire, because the "Next" button on each step is gated. It exists as a backstop for cases where a user navigates directly to confirmation via a deep link or a back-navigation edge case.

**Step 2: Display the order summary**
`ConfirmationStep.tsx`

All three selections are displayed as a review card. Each selection is tappable — tapping routes the user back to that step. The navigation goes directly to the target step index rather than stepping backward sequentially, so tapping "change spread" goes straight to the spread step without resetting bread or jelly state.

**Step 3: Submit the order**
`order-service.ts`

Tapping "Make My Sandwich" calls `submitOrder()` with the full `orderState`. The order service assembles the payload, validates server-side, and returns an order ID. If submission fails, the confirmation screen shows an inline error and the user can retry without losing their selections. On success, `OrderFlowProvider` resets to initial state and the progress screen mounts.

---

*The order flow navigation system separates position (which step you're on) from selections (what you've chosen), persists both in a shared context, and delegates validation to step components — making it straightforward to add or rearrange steps without touching navigation logic.*
