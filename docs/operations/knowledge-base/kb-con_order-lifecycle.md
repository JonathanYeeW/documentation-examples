# The Order Lifecycle

**Created:** 2026-04-11
**Last Updated:** 2026-06-23

---

# Summary

An order in the PB&J system has a lifecycle — it moves through a defined set of states from the moment the user taps "Make My Sandwich" to the moment the sandwich is in their hands (or the order fails). Understanding this lifecycle is important when reasoning about what can go wrong and when, how failures surface to the user, and what "completing an order" actually means in the system.

---

# Lifecycle at a Glance

```
PHASE 1: SUBMITTED
  Order received; nothing physical has happened yet.

PHASE 2: ASSEMBLING
  Ingredients pulled, sandwich built, quality checked.
  → Unrecoverable failure at any step = FAILED

PHASE 3: READY
  Sandwich in pickup area; user notified.

PHASE 4: COLLECTED  (terminal success)
  Sandwich picked up or 5-min timeout fired; metrics logged.

───────────────────────────────────────────────────────
FAILED  (terminal failure — branches from Phase 2)
  Pipeline couldn't recover; user notified, can restart.
```

---

# Phases

The following phases describe the happy path — the sequence an order moves through when everything works. Failure cases are covered separately below.

## Phase 1: Submitted

The user has confirmed their selections and the order has been received by the system. Nothing physical has happened yet — no ingredients have moved, no bread has been sliced. The user sees a progress screen. The system has made a commitment to assemble the sandwich; whether it can fulfill that commitment is determined in the next phase.

This phase is intentionally brief. The transition to `ASSEMBLING` fires as soon as the pipeline handler picks up the order.

## Phase 2: Assembling

Physical assembly is in progress. The order enters this phase the moment ingredients are pulled from storage and exits when the sandwich is plated and the quality photo is taken.

This is the longest-running phase, typically 60–90 seconds, and where most operational work happens. Quality gates fire at the earliest possible point in the pipeline: a failure during ingredient prep wastes nothing, a failure during coverage validation wastes bread and spread. The pipeline fails fast to minimize waste.

## Phase 3: Ready

Assembly is complete. The sandwich is in the pickup area waiting for the user. The system has notified the user — a progress screen update and, if audio is enabled, a chime.

The order stays in this phase until the user collects the sandwich or a 5-minute timeout fires. If the pickup sensor doesn't detect collection within that window, the order transitions to `COLLECTED` anyway. The timeout is a pragmatic concession — we have no reliable mechanism to distinguish "user took it" from "order timed out," and the distinction doesn't matter for reporting purposes.

## Phase 4: Collected

Terminal success state. The sandwich was picked up — or the timeout fired. The order is complete. Metrics are logged: time to serve, selections made, quality check triggers, any prep failures that were recovered silently. No further transitions are possible.

We track the machine and the sandwich, not the person. The order record captures what happened operationally, not what the user experienced.

---

# Failure Cases

The following describes how failures are handled when the happy path cannot be completed.

## Unrecoverable Assembly Failure

Something went wrong during Phase 2 that the pipeline couldn't recover from. The order transitions to `FAILED` — a terminal state. A failed order is a complete record: the failure reason, the step where it occurred, and any ingredients that were wasted are all logged for operational review.

Not every problem during assembly becomes a failure. Coverage gaps trigger a silent touch-up pass. Spread that won't mix gets another stir cycle. These recoverable failures are resolved inside the pipeline and never reach the order state machine — from the lifecycle's perspective, they didn't happen. Only failures the pipeline cannot resolve propagate here.

The user is notified with the specific failure reason (stale ingredient, coverage failure, spread wouldn't mix, etc.) and offered the option to start a new order. The failure message is human-readable and tied to the specific cause — "We couldn't get an even spread on your sandwich" is more useful than "An error occurred."

---

# Design Decisions

The following captures the intentional choices behind how this lifecycle is designed — why things work the way they do, and what this lifecycle is and isn't responsible for.

## What Drives State Transitions

State transitions are event-driven, not time-based. The order doesn't move to `ASSEMBLING` after a delay — it moves when the pipeline handler fires. It doesn't move to `READY` on a timer — it moves when the assembly pipeline emits a completion event. This means state is always an accurate reflection of where the order actually is, not an approximation.

The one exception is the `READY → COLLECTED` timeout, which is time-based because the pickup sensor can't distinguish collection from abandonment. Every other transition is triggered by a real event in the system.

## What This Lifecycle Is Not Responsible For

The order lifecycle tracks what happened to an order — not what the user did. Which screen they were on, how long they spent selecting, whether they abandoned mid-flow — that's session state, not order state.

It also doesn't track the internal detail of assembly. Whether coverage passed on the first check or required a touch-up, whether toasting took 30 seconds or 45 — none of that is in the order state machine. It lives in the assembly log, a separate operational record. The lifecycle only needs to know: did assembly succeed or fail?
