# The Order Lifecycle

**Created:** 2026-04-11
**Last Updated:** 2026-04-11

An order in the PB&J system has a lifecycle — it moves through a defined set of states from the moment the user taps "Make My Sandwich" to the moment the sandwich is in their hands (or the order fails). Understanding this lifecycle is important when reasoning about what can go wrong and when, how failures surface to the user, and what "completing an order" actually means in the system.

## The States

```
SUBMITTED → ASSEMBLING → READY → COLLECTED
               ↓
            FAILED
```

An order enters only one of two terminal states: `COLLECTED` (success) or `FAILED` (something went wrong). Everything before is transient.

**SUBMITTED** — The user has confirmed their selections and the order has been received by the system. Nothing physical has happened yet. The user sees a progress screen. The system has made a commitment to assemble the sandwich; whether it can fulfill that commitment is determined in the next state.

**ASSEMBLING** — Physical assembly is in progress. The order is in this state from the moment ingredients are pulled from storage until the sandwich is plated and the quality photo is taken. This is the longest-running state, typically 60–90 seconds. An order can fail from this state if any quality gate fails and cannot be recovered. Most operational failures happen here.

**READY** — Assembly is complete and the sandwich is in the pickup area waiting for the user. The system has notified the user. The order stays in this state until the user takes the sandwich or a timeout occurs. If the pickup sensor doesn't detect collection within 5 minutes, the order transitions to `COLLECTED` anyway — we don't have a mechanism to distinguish between "user took it" and "order timed out," and it doesn't matter for reporting purposes.

**COLLECTED** — Terminal success state. The sandwich was picked up (or timed out). The order is complete. Metrics are logged. No further transitions.

**FAILED** — Terminal failure state. Something went wrong during assembly that couldn't be recovered. The user is notified with the specific failure reason (stale ingredient, coverage failure, etc.) and offered the option to start a new order. A failed order is a complete record — the failure reason, the step where it occurred, and any ingredients that were wasted are all logged for operational review.

## What Drives State Transitions

State transitions are not time-based — they're event-based. The order doesn't move to `ASSEMBLING` after a delay; it moves when the pipeline handler fires. It doesn't move to `READY` after a timer; it moves when the assembly pipeline emits a completion event.

This matters because it means state is always an accurate reflection of where the order actually is in the physical process, not an approximation. When you see `state: ASSEMBLING` in the order record, assembly is genuinely in progress — not "probably in progress based on timing."

The one exception is the `READY → COLLECTED` timeout. That's time-based because we have no reliable sensor for "user took the sandwich." It's a pragmatic concession, not a design preference.

## How Failures Work

A failure can occur at any point during `ASSEMBLING`. The failure model is intentionally simple: if a failure is unrecoverable, the order fails immediately and completely. There is no partial success state — an order is either good or it isn't.

Recoverable failures (like a coverage gap that can be fixed with a touch-up pass) are handled silently inside the pipeline and never reach the order state machine. From the order lifecycle's perspective, they didn't happen. Only failures that cannot be resolved within the pipeline propagate to `FAILED`.

This means `FAILED` is a reliable signal: if you see it, the pipeline tried and couldn't recover. You don't need to investigate whether a "soft failure" happened — soft failures don't exist in the order record.

## What the Lifecycle Is Not Responsible For

The order lifecycle tracks what happened to an order. It doesn't track what the user did — which screen they were on, how long they spent selecting, whether they abandoned mid-flow. That's session state, not order state.

It also doesn't track the quality of the assembly in detail. Whether coverage passed on the first check or required a touch-up, whether toasting took 30 seconds or 45 — none of that is in the order state machine. It lives in the assembly log, which is a separate operational record. The order lifecycle only needs to know: did assembly succeed or fail?
