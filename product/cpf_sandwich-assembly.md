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

**Step 1: Tap to start.** The machine sits idle on a welcome screen. Tap anywhere to begin. No login, no account — the machine serves whoever is standing in front of it.

**Step 2: Bread selection.** Available bread types shown as selectable cards with short descriptions. If the machine supports toasting, a toggle appears below the selection, defaulted to off.

**Step 3: Spread selection.** Available spreads shown as selectable cards. Each card shows user-facing notes from the Spread Registry — things like "This spread is natural and will be stirred fresh" for almond butter. The notes set expectations about prep time without exposing system internals.

**Step 4: Jelly selection.** Same card pattern as spread. No prep notes — jelly doesn't have handling that affects the user's wait.

**Step 5: Order confirmation.** One screen showing all selections with a "Make My Sandwich" button. Tap any selection to go back and change it. This is the last moment before ingredients start moving.

**Step 6: Order queued.** The screen transitions to a progress view showing which phase the machine is in. The user waits here through Phase 2.

---

## Phase 2: Assembly

Machine-driven. The user sees a progress indicator but can't intervene. Every quality gate fires as early as possible — a failed order at step 7 wastes nothing, a failed order at step 10 wastes ingredients and time.

**Step 7: Ingredient preparation.** The machine resolves the order into concrete items, pulls them from storage, and stages them at the prep station. Quality checks run on every ingredient — stale bread, expired jelly, and dried-out spread get caught here. If anything fails, the order stops before assembly begins.

**Step 8: Bread handling.** Sliced if needed, toasted if requested, laid out for assembly. Toasted bread gets a mandatory 30-second cool-down before spread application — without it, spread melts into the bread grain and the sandwich falls apart on first bite (see INC-042).

**Step 9: Spread pre-handling.** The machine checks the Spread Registry for handling requirements. Natural spreads get stirred (up to 3 cycles). Refrigerated spreads get tempered to room temperature. Standard peanut butter skips this step. If prep fails after max retries, the order fails with a user-friendly message and the user picks a different spread.

**Step 10: Assembly.** Spread on Slice A, jelly on Slice B, coverage validation, combine, cut (diagonal default), plate. Coverage validation catches bare spots larger than 1cm² — a failed check triggers one touch-up pass, then fails the order if coverage still isn't clean.

**Step 11: Quality check and completion.** A photo of the finished sandwich is taken for the order record. Order marked complete.

---

## Phase 3: Delivery

The handoff back to the user.

**Step 12: User notified.** The progress screen updates to show the sandwich is ready. If the machine has audio, a chime plays.

**Step 13: Sandwich dispensed.** The sandwich slides into the pickup area. The user takes it. The welcome screen returns after 30 seconds of inactivity.

**Step 14: Metrics logged.** The system records the completed order — time to serve, selections, quality check triggers, prep failures. This data feeds operational dashboards. We track the machine and the sandwich, not the person.
