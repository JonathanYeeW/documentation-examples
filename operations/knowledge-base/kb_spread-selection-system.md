# Spread Selection System

**Created:** 2026-03-11
**Last Updated:** 2026-03-11
**Directory:** `sandwich-service/spread-selector`

The spread selection system lets users choose a spread — creamy peanut butter, crunchy peanut butter, almond butter, or sunflower butter — as part of building their sandwich order. It covers how the selector UI works, how selections persist across order flow navigation, how availability is enforced, and how the selection is handed off to the assembly pipeline.

Read this when you're working on the spread selector component, the order flow state model, or anything in the assembly pipeline that touches spread type. The spread selection state bug documented in ENG-312 provides important context on why the shared state pattern here is the way it is.

## Part 1: 🥜 The Spread Selector

The spread selector is the UI step where users make their selection. It sits between the jelly step and the confirmation step in the order flow. Each step in the flow is a separate component — when a user navigates away, the component unmounts and local state is destroyed. This means any selection that needs to survive navigation must be written to shared order state, not held locally.

**Step 1:** **Render available spreads**
`SpreadSelector.tsx`

On mount, the component fetches the current spread availability list from `spread-availability-service`. Only spreads with `status: available` render as selectable cards. Out-of-stock spreads render as disabled cards with a tooltip — visible but unselectable. We show disabled options rather than hiding them so users know what the machine supports; hiding them creates confusion when a spread reappears after a restock.

**Step 2:** **Handle selection**
`SpreadSelector.tsx`

When a user selects a spread, the component updates the visual state and writes the value to `orderState.spreadType`. Writing to shared state on every selection change — not only on "Next" — ensures the selection survives navigation. If you move the write to happen only at step completion, back-navigation will lose the selection.

**Step 3:** **Restore selection on remount**
`SpreadSelector.tsx`

When the user navigates back to this step, the component reads `orderState.spreadType` on mount and restores the selected card. If `orderState.spreadType` is null, no card is highlighted and the user must select before continuing. This is the pattern all step components follow — `BreadSelector` and `JellySelector` use the same read-on-mount behavior against their respective fields in order state.

- **If a selection exists in shared state**: Restore it and render as active
- **If no selection exists**: Render all available cards unselected

## Part 2: 📦 Spread Availability

Not all spreads are always available. Availability is managed centrally so changes take effect immediately without a deploy.

**Step 1:** **Fetch availability on mount**
`spread-availability-service.ts`

`SpreadSelector` calls `getAvailableSpreads()` on every mount. A spread is unavailable if inventory stock is low OR a manual operations override is active — either condition is sufficient. Fetching on every mount rather than caching means a user who navigates away and returns gets a fresh availability check. This matters during high-traffic periods when spreads can sell out mid-session.

**Step 2:** **Handle availability changes mid-session**
`spread-availability-service.ts`

If a user's selected spread becomes unavailable between step selection and order confirmation — because stock ran out or operations toggled an override — the confirmation step surfaces an error and routes the user back to the spread step to reselect. The selector remounts, fetches fresh availability, and shows the previously selected spread as disabled with an explanatory message. Don't cache availability at the session level; this re-check on remount is what makes the mid-session case work.

**Step 3:** **Operations override**
`spread-config.ts`

Operations can manually mark a spread unavailable through the admin panel without touching inventory counts. Overrides are used for maintenance windows, quality holds, or supply issues where the inventory count is technically non-zero but the spread shouldn't be offered. Manual overrides take precedence over inventory status. Overrides must be lifted manually — they don't expire automatically.

## Part 3: 🔗 Handoff to Assembly

Once the user confirms their order, spread selection moves from order state into the assembly pipeline.

**Step 1:** **Include spreadType in order payload**
`order-service.ts`

At confirmation, the order service assembles the full payload from `orderState`. `spreadType` is a required field — orders missing it are rejected before anything reaches the assembly pipeline, and the user is routed back to the spread selector. This validation exists because early in the order flow it was possible to reach confirmation with a null spread type if the user navigated around the spread step; the hard rejection here is the backstop.

**Step 2:** **Map selection to assembly instructions**
`order-resolver.ts`

The order resolver translates `spreadType` from a user-facing label (e.g., `"sunflower_butter"`) to an assembly instruction set: the specific inventory SKU, storage location, and applicator pressure setting for that spread. Creamy peanut butter uses light applicator pressure. Crunchy uses medium pressure with an extra distribution pass. Almond butter and sunflower butter both default to the creamy pressure profile. If a new spread type is added to the catalog, a corresponding mapping must be added here — missing entries cause the resolver to throw at assembly time.

**Step 3:** **Pass to spread applicator**
`spread-applicator.ts`

The resolved spread instructions are passed to `spread-applicator` as part of the full assembly job. From this point, spread type is an assembly concern. The applicator handles quantity, motion pattern, and coverage validation using the instruction set from the resolver.

---

*The spread selection system runs across three surfaces: the selector UI where users pick and persist their spread, the availability layer that enforces stock and operational rules in real time, and the order handoff that translates a selection into assembly instructions.*
