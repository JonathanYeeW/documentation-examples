# Core Product Flow — Order to Sandwich

**Created:** 2026-03-01
**Last Updated:** 2026-03-01
**Context:** End-to-end flow from a hungry kid to a finished sandwich in their hands.

## Summary

This document covers the complete user journey through the PB&J Machine — from the moment someone approaches the machine to the moment they're holding a sandwich. The flow has three phases: the user tells the machine what they want, the machine builds it, and the sandwich is delivered. The entire interaction takes under two minutes. Understanding this flow is essential before working on any feature — every change we make touches some part of this path.

## Flow at a Glance

```
PHASE 1: ORDER (user-driven, 15–20 seconds)

  1. User taps to start
  2. Select bread + optional toasting
  3. Select spread
  4. Select jelly
  5. Review selections → confirm
  6. Order queued for assembly

        ~~~~~~~~ handoff to machine ~~~~~~~~

PHASE 2: ASSEMBLY (machine-driven, 60–90 seconds)

  7.  Ingredient prep — pull, check quality, stage
  8.  Bread handling — slice, toast, cool-down
  9.  Spread pre-handling — stir or temper per config
  10. Assembly — spread, jelly, validate, combine, cut, plate
  11. Quality photo → order complete

      → Quality gate failure at any step = order fails,
        user notified, restart from Phase 1

        ~~~~~~~~ handoff to user ~~~~~~~~

PHASE 3: DELIVERY (5 seconds)

  12. User notified — chime plays
  13. Sandwich dispensed — user takes it
  14. Metrics logged, welcome screen returns
```

---

## Phase 1: Order

User-driven. A guided wizard walks the user through their selections one choice per screen.

**Step 1: Tap to start.** The machine sits idle on a welcome screen until someone touches it.

- Tap anywhere to begin.
- No login and no account — the machine serves whoever is standing in front of it.

**Step 2: Bread selection.** The first of three choices, each on its own screen.

- Available bread types show as selectable cards with short descriptions.
- A toasting toggle appears below the selection when the machine supports it, defaulted to off.

**Step 3: Spread selection.** The same card pattern as bread, and the only place the Spread Registry surfaces to the user.

- Available spreads show as selectable cards.
- Each card carries user-facing notes from the registry — "This spread is natural and will be stirred fresh" for almond butter.
- **Why the notes exist** — they set expectations about prep time without exposing system internals.

**Step 4: Jelly selection.** The same card pattern as spread, with no prep notes — jelly has no handling that affects the user's wait.

**Step 5: Order confirmation.** The last moment before ingredients start moving.

- One screen shows every selection, with a "Make My Sandwich" button.
- Tapping any selection goes back to change it.

**Step 6: Order queued.** The handoff from user to machine.

- The screen transitions to a progress view showing which phase the machine is in.
- The user waits here through Phase 2.

---

## Phase 2: Assembly

Machine-driven. The user watches a progress screen and makes no further decisions.

**Step 7: Ingredient preparation.** The order becomes physical items, and this is the cheapest place to stop.

- The machine resolves the order into concrete items, pulls them from storage, and stages them at the prep station.
- Quality checks run on every ingredient — stale bread, expired jelly, dried-out spread.
- A failure here stops the order before assembly begins.

**Step 8: Bread handling.** Slice, toast, lay out.

- Sliced if needed, toasted if requested, laid out for assembly.
- Toasted bread gets a mandatory 30-second cool-down before spread application.
- **INC-042** — without the cool-down, spread melts into the bread grain and the sandwich falls apart on first bite.

**Step 9: Spread pre-handling.** Conditional, driven by the Spread Registry rather than by the order.

- Natural spreads get stirred, up to 3 cycles.
- Refrigerated spreads get tempered to room temperature.
- Standard peanut butter skips this step entirely.
- A failure after max retries fails the order with a user-friendly message, and the user picks a different spread.

**Step 10: Assembly.** The fixed sequence every order runs through.

- Spread on Slice A, jelly on Slice B.
- Coverage validation catches bare spots larger than 1cm².
- A failed check triggers one touch-up pass, then fails the order if coverage still is not clean.
- Combine, cut — diagonal by default — and plate.

**Step 11: Quality check and completion.** A photo of the finished sandwich is taken for the order record, and the order is marked complete.

---

## Phase 3: Delivery

The handoff back to the user.

**Step 12: User notified.** The machine signals that it is done.

- The progress screen updates to show the sandwich is ready.
- A chime plays when the machine has audio.

**Step 13: Sandwich dispensed.** The end of the path.

- The sandwich slides into the pickup area and the user takes it.
- The welcome screen returns after 30 seconds of inactivity.

**Step 14: Metrics logged.** What the system keeps, and what it deliberately does not.

- Recorded: time to serve, selections, quality check triggers, prep failures.
- The data feeds operational dashboards.

The machine and the sandwich are tracked, not the person.
