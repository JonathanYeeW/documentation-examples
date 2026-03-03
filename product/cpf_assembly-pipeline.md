# Core Product Flow — Assembly Pipeline

**Created:** 2026-03-02
**Last Updated:** 2026-03-02
**Context:** End-to-end flow from a confirmed order entering the assembly function to a plated sandwich returned or an order failure. This is the orchestration layer inside `assembleSandwich`, covering spread resolution, conditional pre-handling, the core assembly sequence, and quality checks.

## Summary

This document covers the complete assembly pipeline — from the moment a confirmed order calls `assembleSandwich(spreadType, breadType, jellyFlavor)` to the moment a plated sandwich is returned or the order fails. The flow has four phases: the function resolves the spread selection against the registry, runs conditional pre-handling based on spread properties, executes the fixed assembly sequence, and performs final quality checks before returning. The entire pipeline is synchronous — the caller waits for completion. Understanding this flow is essential before adding new spread types, modifying the assembly sequence, or changing quality gate behavior.

## Flow at a Glance

```
PHASE 1: SPREAD RESOLUTION (registry lookup)

  1. Look up spread config from registry
  2. Validate spread is known and available

        ~~~~~~~~ handoff to pre-handling ~~~~~~~~

PHASE 2: PRE-HANDLING (conditional, spread-driven)

  3.  If spread requires stirring → stir until consistent
  4.  If spread requires tempering → temper to target temp

      → Pre-handling failure = order fails before
        any ingredients are consumed

        ~~~~~~~~ handoff to assembly ~~~~~~~~

PHASE 3: ASSEMBLY (fixed sequence)

  5.  Prepare bread — slice, toast if requested, cool-down
  6.  Apply spread — pressure adjusted by viscosity
  7.  Apply jelly
  8.  Coverage validation on both slices
  9.  Combine, cut, plate

        ~~~~~~~~ handoff to quality ~~~~~~~~

PHASE 4: QUALITY & RETURN

  10. Quality photo for order record
  11. Return plated sandwich

      → Quality gate failure at any assembly step =
        order fails, ingredients wasted, user notified
```

---

## Phase 1: Spread Resolution

Registry-driven. The function resolves the user's spread selection into a concrete configuration before touching any ingredients. Failures here are cheap — nothing has been opened, nothing is wasted.

**Step 1: Look up spread config.** The function calls `getSpreadConfig(spreadType)` with the spread key from the user's order (e.g., "almond_butter"). The registry returns the full `SpreadConfig` — label, viscosity, and the boolean flags that drive pre-handling (stir, refrigerate). This is the only integration point between the pipeline and the registry. Everything downstream reads from the resolved config, not the raw spread key.

**Step 2: Validate spread.** If the spread key isn't in the registry, `getSpreadConfig` throws `UnknownSpreadError` with the list of valid spread types included in the error. This is a developer error — the UI should only offer spreads that exist in the registry. The order fails immediately and no ingredients are pulled.

---

## Phase 2: Pre-Handling

Conditional. The spread config's boolean flags determine which pre-handling steps run. Standard peanut butter skips this phase entirely — no stir, no temper. Natural almond butter runs both. Each pre-handling step prepares the spread in-place before assembly begins. Failures here stop the order before any bread or jelly is consumed.

**Step 3: Stir (if required).** For natural spreads that separate over time, the pipeline opens the container and stirs in 30-second cycles. After each cycle, consistency is checked — if the oil is still separated, another cycle runs. Maximum three cycles. If the spread won't mix after three rounds, the container is likely too old or too cold, and the order fails with `SpreadNotMixedError`. The error suggests a replacement container so operations can act on it.

**Step 4: Temper (if required).** For refrigerated spreads, the pipeline removes the container from cold storage and lets it rest at room temperature. Temperature is checked every 60 seconds. Once within 5°F of the target (65°F), the spread is ready. Exact temperature doesn't matter — close enough means soft enough to apply without tearing bread. If tempering times out after 10 minutes, the pipeline proceeds with a warning flag rather than failing the order. A slightly cold spread may affect consistency but won't break anything. The warning is logged so operations can track if specific spreads consistently fail to temper in time.

---

## Phase 3: Assembly

Fixed sequence. Every order runs through the same steps in the same order. The spread config influences spreader pressure (step 6), but the sequence itself doesn't change. Quality gates fire at the earliest possible point — a failure at step 5 wastes bread but saves spread and jelly, a failure at step 8 wastes everything.

**Step 5: Prepare bread.** Bread is sliced if needed and toasted if the user requested it. Toasted bread gets a mandatory 30-second cool-down before spread application. Without the cool-down, spread melts into the bread grain and the sandwich falls apart on first bite. This was discovered in production (INC-042) and is non-negotiable — the pipeline blocks until cool-down completes regardless of order throughput pressure.

**Step 6: Apply spread.** Spread goes on Slice A. The spreader tool adjusts pressure based on the `viscosity` field from the spread config — thick spreads need more force, thin spreads need a lighter touch. If the bread is still warm from toasting, the pipeline reduces spread quantity by 15% to prevent sogginess from accelerated melting. If the spread is too viscous to apply (can happen with under-tempered refrigerated spreads), the pipeline warms the spread for 10 seconds and retries once. A second failure throws `SpreadApplicationError` and the order fails. No partial sandwich is served.

**Step 7: Apply jelly.** Jelly goes on Slice B. Straightforward application — jelly doesn't have the viscosity variance or temperature sensitivity that spreads do. No conditional logic, no pre-handling.

**Step 8: Coverage validation.** Both slices are scanned for bare spots larger than 1cm². This catches uneven spread application, jelly pooling in one corner, or spots where the bread texture prevented adhesion. A failed check triggers one touch-up pass on the offending slice. If coverage still isn't clean after the touch-up, the order fails. This is the last quality gate before the sandwich becomes irreversible — once the slices are combined, there's no fixing a coverage problem.

**Step 9: Combine, cut, plate.** Slice A and Slice B are pressed together, cut diagonally (default), and placed on a plate. This step is mechanical and has no conditional logic or quality gates — if we got here, the sandwich is good.

---

## Phase 4: Quality & Return

The final check and handoff back to the caller.

**Step 10: Quality photo.** A photo of the plated sandwich is taken for the order record. This serves the operations dashboard, not the user — it's evidence that the sandwich met quality standards at the moment of completion. The photo is stored with the order metadata.

**Step 11: Return.** The function returns the plated sandwich. Same output shape regardless of which spread was selected or which pre-handling steps ran — callers don't need to know what happened inside the pipeline. If any step in Phases 2–3 threw, the error propagates to the caller with enough context to notify the user and suggest a retry or different selection.

**Error handling.** Failures fall into two categories based on when they happen. Pre-assembly failures (Phase 1–2) are cheap — no ingredients consumed, the user is asked to select a different spread or try again. Assembly failures (Phase 3) are expensive — bread and potentially spread are wasted, but the alternative is serving a bad sandwich. The pipeline never serves a partial or substandard sandwich. Every failure mode prefers wasting ingredients over delivering a bad experience.
