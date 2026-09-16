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

Registry-driven. The function resolves the user's spread selection into a concrete configuration before touching any ingredients, so failures here are cheap — nothing has been opened and nothing is wasted.

**Step 1: Look up spread config.** The only integration point between the pipeline and the registry.

- `getSpreadConfig(spreadType)` is called with the spread key from the order, such as `almond_butter`.
- It returns the full `SpreadConfig` — label, viscosity, and the boolean flags that drive pre-handling.
- Everything downstream reads the resolved config, never the raw spread key.

**Step 2: Validate spread.** The guard against a key the registry does not hold.

- An unknown key throws `UnknownSpreadError`, with the list of valid spread types in the error.
- The order fails immediately and no ingredients are pulled.
- **This is a developer error** — the UI should only offer spreads that exist in the registry.

---

## Phase 2: Pre-Handling

Conditional. The spread config's boolean flags decide which steps run — standard peanut butter skips the phase entirely, natural almond butter runs both. Failures here stop the order before any bread or jelly is consumed.

**Step 3: Stir, if required.** For natural spreads that separate over time.

- The container is opened and stirred in 30-second cycles, with consistency checked after each.
- Separated oil triggers another cycle, to a maximum of three.
- A spread that will not mix after three rounds fails the order with `SpreadNotMixedError`, which names a replacement container so operations can act on it.

**Step 4: Temper, if required.** For refrigerated spreads.

- The container is removed from cold storage and rests at room temperature, with the temperature checked every 60 seconds.
- Within 5°F of the 65°F target counts as ready. Exact temperature does not matter — close enough means soft enough to apply without tearing bread.
- A timeout after 10 minutes proceeds with a warning flag rather than failing the order, because a slightly cold spread may affect consistency but will not break anything.
- The warning is logged so operations can track spreads that consistently fail to temper in time.

---

## Phase 3: Assembly

Fixed sequence. Every order runs the same steps in the same order; the spread config influences spreader pressure at step 6 but never the sequence itself. Quality gates fire at the earliest possible point, because a failure at step 5 wastes bread while a failure at step 8 wastes everything.

**Step 5: Prepare bread.** Slice, toast, cool.

- Bread is sliced if needed and toasted if the user requested it.
- Toasted bread gets a mandatory 30-second cool-down before spread application.
- **INC-042** — without the cool-down, spread melts into the bread grain and the sandwich falls apart on first bite. The pipeline blocks until it completes regardless of throughput pressure.

**Step 6: Apply spread.** Spread goes on Slice A, at a pressure the config decides.

- The spreader adjusts pressure from the `viscosity` field — thick spreads need more force, thin spreads a lighter touch.
- Bread still warm from toasting reduces spread quantity by 15%, to prevent sogginess from accelerated melting.
- A spread too viscous to apply is warmed for 10 seconds and retried once. A second failure throws `SpreadApplicationError`.
- No partial sandwich is ever served.

**Step 7: Apply jelly.** Jelly goes on Slice B, with no conditional logic and no pre-handling — it has none of the viscosity variance or temperature sensitivity that spreads do.

**Step 8: Coverage validation.** The last quality gate before the sandwich becomes irreversible.

- Both slices are scanned for bare spots larger than 1cm², catching uneven spread, pooled jelly, and spots where bread texture prevented adhesion.
- A failed check triggers one touch-up pass on the offending slice.
- Coverage still unclean after the touch-up fails the order.
- Once the slices are combined there is no fixing a coverage problem, which is why this gate sits here.

**Step 9: Combine, cut, plate.** Slices are pressed together, cut diagonally by default, and placed on a plate. No conditional logic and no quality gates — if the pipeline got here, the sandwich is good.

---

## Phase 4: Quality & Return

The final check and the handoff back to the caller.

**Step 10: Quality photo.** A photo of the plated sandwich is taken for the order record.

- It serves the operations dashboard, not the user — evidence that the sandwich met quality standards at the moment of completion.
- Stored with the order metadata.

**Step 11: Return.** The function returns the plated sandwich.

- The output shape is the same regardless of which spread was selected or which pre-handling ran. Callers do not need to know what happened inside.
- An error thrown anywhere in Phases 2–3 propagates with enough context to notify the user and suggest a retry or a different selection.

---

## Error Handling

Failures fall into two classes, and the class is decided by when the failure happens rather than by what went wrong.

| Class | Phases | What it costs | What the user is told |
|---|---|---|---|
| Pre-assembly | 1–2 | Nothing. No ingredients consumed. | Select a different spread, or try again. |
| Assembly | 3 | Bread, and potentially spread. | The order failed; retry or change the selection. |

The pipeline never serves a partial or substandard sandwich. Every failure mode prefers wasting ingredients over delivering a bad experience.
