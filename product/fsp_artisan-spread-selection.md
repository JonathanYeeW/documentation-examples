I # Artisan Spread Selection

**Created:** 2026-03-01
**Last Updated:** 2026-03-01 (Session 1)
**Context:** Evolving PB&J Assembly from fixed peanut butter to user-selectable spread options

## 📋 Summary

Our system currently only supports peanut butter, which means users who want a different spread have to make their sandwich outside the system entirely. Artisan Spread Selection lets users pick their spread when they order. The system knows how to handle each spread differently and applies the right preparation automatically. The core assembly flow stays the same — we're just opening up the spread step to support multiple options.

## 👤 User Perspective

These two flows show the user's experience before and after this feature. Each step is a single action so you can scan both flows and see exactly where the new behavior slots in.

### Current

```
1. User opens PB&J Builder
2. Select bread type
3. Select jelly flavor
4. Confirm order
5. System assembles with peanut butter (no choice offered)
6. User receives sandwich
```

### With This Feature

```
1. User opens PB&J Builder
2. Select bread type
3. Select jelly flavor
4. Select spread from available options
5. System shows spread-specific notes (e.g., "Stir before use" for natural almond butter)
6. Confirm order
7. System assembles with chosen spread, applying spread-specific handling
8. User receives sandwich
```

## 🔧 Technical Overview

The assembly pipeline currently hardcodes peanut butter as the only spread. We're introducing a Spread Registry — a config file that maps each spread type to its handling requirements — and updating the pipeline to read from the registry instead. The pipeline changes in one place: the spread step queries the registry with the user's selection instead of using a constant. Everything else (bread handling, jelly, cutting, plating) stays the same.

## 🏗️ Implementation

```
Step 1: Create the Spread Registry
Step 2: Update the Assembly Pipeline to accept spread selection
Step 3: Add spread-specific pre-handling (stir, temper)
Step 4: Update the UI to show spread options
```

### Step 1: Create the Spread Registry

The registry is a static configuration file that maps spread identifiers to their handling properties. We're keeping this simple — no database, no admin UI. Adding a new spread means adding a line to the config.

```
spread-registry.ts — SpreadRegistry:

  Input: None — this is a static config file, not a runtime call.

  // ---- Registry Definition ----
  1. Define SpreadConfig type: { label, refrigerate, stir, viscosity }
  2. Export SPREAD_REGISTRY as Record<string, SpreadConfig>
  3. Populate with initial three spreads:
     → "peanut_butter": thick, no stir, no refrigerate
     → "almond_butter": medium, stir required, refrigerate
     → "sunflower_butter": thick, no stir, no refrigerate

  // ---- Lookup ----
  4. getSpreadConfig(spreadType: string) → SpreadConfig
  5. If spreadType not in registry → throw UnknownSpreadError
     → Include available spread types in the error so the caller
       knows what's valid without checking the registry themselves.
  6. Return config

  Output: SpreadRegistry module exporting:
    - SPREAD_REGISTRY: Record<string, SpreadConfig>
    - getSpreadConfig(spreadType) → SpreadConfig
```

**Error Handling**

```
  Unknown spread type requested:
    → Throw UnknownSpreadError with available spread types listed.
    → This is a developer error (bad data from UI) not a user error.

  Registry file malformed:
    → Fail at startup with clear error message, not at assembly time.
    → If the registry is broken, we want to know immediately —
      not when the first sandwich order comes in.
```

**Edge Cases**

```
  Empty string as spreadType:
    → Treat as missing selection, prompt user to choose.
    → Don't pass empty string to the registry — validate upstream.

  Spread type exists in registry but is out of stock:
    → Not handled in this feature (future inventory system concern).
    → Registry answers "how to handle this spread," not "do we have it."
```

### Step 2: Update the Assembly Pipeline

The assembly function currently takes `(breadType, jellyFlavor)`. We add `spreadType` as the first parameter and query the registry at the start of assembly.

```
assembly-pipeline.ts — assembleSandwich(spreadType, breadType, jellyFlavor):

  Input: {
    spreadType: string,    // user's spread selection from the UI
    breadType: string,     // existing param, unchanged
    jellyFlavor: string    // existing param, unchanged
  }

  // ---- Spread Resolution ----
  1. spreadConfig = SpreadRegistry.getSpreadConfig(spreadType)
     → This is the only new line in the pipeline. Everything
       below already existed — we're just swapping the constant
       for a registry lookup.
  
  // ---- Pre-Handling (NEW) ----
  2. If spreadConfig.stir → call stirSpread(spreadType)
     → See Step 3 for stirSpread implementation.
  3. If spreadConfig.refrigerate → call temperSpread(spreadType, targetTemp: 65°F)
     → See Step 3 for temperSpread implementation.
  
  // ---- Assembly (existing logic, unchanged) ----
  4. prepareBread(breadType)
  5. applySpread(spreadConfig)        // was: applyPeanutButter()
     → applySpread now reads viscosity from the config to determine
       spreader pressure. Previously hardcoded to "thick."
  6. applyJelly(jellyFlavor)
  7. combineSandwich()
  8. cut()
  9. plate()

  Output: { sandwich }
    → A fully assembled, cut, and plated sandwich.
    → Same output shape as before — callers don't need to change.
```

**Error Handling**

```
  Spread too viscous to apply (step 5):
    → Warm spread 10 seconds, retry once, then fail with SpreadApplicationError.
    → Don't serve a partial sandwich — if the spread won't apply,
      the order fails here.

  Spread container empty mid-application (step 5):
    → Fail with OutOfSpreadError.
    → Same principle: don't serve a partial sandwich.
    → Bread is wasted, but that's better than a bad user experience.
```

**Edge Cases**

```
  Natural almond butter that has separated:
    → stirSpread() in Step 3 handles this before we get to application.
    → If stir didn't fully mix, viscosity check at step 5 catches it.

  Sunflower butter on warm bread (bread just toasted):
    → Spread melts faster than expected on hot bread.
    → Reduce spread amount by 15% to prevent sogginess.
    → This is a known interaction — the pipeline checks bread
      temperature against spread melt point before applying.
```

### Step 3: Add Spread-Specific Pre-Handling

New functions that the assembly pipeline calls based on spread config flags. These don't exist today because peanut butter needed neither. Both are called from Step 2 before assembly begins.

```
spread-handlers.ts — stirSpread(spreadType):

  Input: {
    spreadType: string    // which spread we're preparing
  }

  // ---- Stir ----
  1. Open spread container
  2. Insert stir tool
  3. Stir in circular motion for 30 seconds
  4. Check consistency — if still separated, stir 15 more seconds
     → Natural nut butters separate over time. Oil rises to the top.
       Stirring re-incorporates it into a spreadable consistency.
  5. Max 3 stir cycles, then fail with SpreadNotMixedError
     → If 3 rounds of stirring can't fix it, the container is
       likely too old or too cold. Don't keep trying.

  Output: void
    → No return value. Prepares the spread in-place.
    → If preparation fails, throws — the pipeline catches it.
```

```
spread-handlers.ts — temperSpread(spreadType, targetTemp):

  Input: {
    spreadType: string,    // which spread we're tempering
    targetTemp: number     // target temperature in °F
  }

  // ---- Temper ----
  1. Remove spread from refrigerator
  2. Let rest at room temperature
  3. Check temp every 60 seconds
  4. When within 5°F of target → ready
     → We don't need exact temperature — close enough means
       the spread is soft enough to apply without tearing bread.
  5. Timeout after 10 minutes → proceed anyway with warning flag
     → Some spreads are slow to temper. Rather than blocking
       the order indefinitely, we proceed with a warning.
       May affect spread consistency but won't break anything.

  Output: void
    → No return value. Tempers the spread in-place.
    → On timeout, sets a warning flag but does not throw.
```

**Error Handling**

```
  Spread won't mix after 3 cycles (stirSpread step 5):
    → Throw SpreadNotMixedError.
    → Suggest replacement container in the error message.
    → Pipeline catches this and fails the order.

  Spread won't reach target temp (temperSpread step 5):
    → Don't throw — proceed with warning flag.
    → Log the warning so we can track if specific spreads
      consistently fail to temper in time.
```

### Step 4: Update the UI

The selection screen adds a spread picker before the existing bread and jelly selectors. Each option shows the spread name and any user-facing notes from the config.

```
spread-selector.tsx — SpreadSelector:

  Input: None — component fetches its own data from the registry.

  // ---- Load Options ----
  1. Fetch available spreads from registry
     → Returns all spread configs. No filtering for now —
       every spread in the registry is available to select.

  // ---- Render ----
  2. Render as selectable cards with label and notes
  3. If spread.stir → show note: "This spread is natural and will be stirred fresh"
  4. If spread.refrigerate → show note: "Chilled for freshness, may take a moment to prep"
     → Notes are user-facing translations of the config flags.
       The user doesn't see "stir: true" — they see a friendly note.

  // ---- Selection ----
  5. On selection → pass spreadType to assembly flow
     → spreadType is the registry key (e.g., "almond_butter"),
       not the display label.

  Output: spreadType: string
    → The user's selection, passed to the assembly flow on confirm.
```

**Error Handling**

```
  Registry returns empty list (step 1):
    → Show "No spreads available" with contact support link.
    → This shouldn't happen in practice — means the registry
      config is empty or broken.

  User proceeds without selecting (step 5):
    → Block submission, highlight spread selector.
    → spreadType is required. Don't fall back to a default.
```

## 🗑️ Removing a Spread from the Registry

If a spread is discontinued or a supplier drops out, we need to remove it without breaking in-flight orders. This operation only exists because the registry exists — it's a maintenance concern, not part of the core assembly flow.

```
spread-registry.ts — removeSpread(spreadType):

  // ---- Check for Active Orders ----
  1. Query order system for any in-progress orders using this spreadType
  2. If active orders exist → block removal, return list of order IDs

  // ---- Remove ----
  3. Delete spread entry from registry config
  4. Log removal with timestamp and reason

  // ---- Verify ----
  5. Confirm getSpreadConfig(spreadType) now throws UnknownSpreadError
  6. Confirm UI no longer shows the removed spread as an option
```

**Error Handling**

```
  Active orders using this spread (step 2) → block removal, surface order IDs so they can be resolved first
  Registry file write failure (step 3) → no change applied, spread remains available, safe to retry
```

**Edge Cases**

```
  Last spread in registry:
    → Don't allow removal. System must have at least one spread available.
    → Return error: "Cannot remove the only available spread."
  Spread removed while user is mid-order:
    → User selected this spread but hasn't confirmed yet.
    → On confirmation, registry lookup fails → show "This spread is no longer available, please select another."
```

## 📦 Data Models

```
SpreadConfig (NEW) {
  label: string           // Display name — "Almond Butter"
  refrigerate: boolean    // Needs cold storage, requires tempering before use
  stir: boolean           // Natural spread that separates, needs stirring
  viscosity: "thin" | "medium" | "thick"  // Determines spreader tool pressure
}
```

## ❓ FAQ

1. **Why a static registry instead of a database table?** We have three spreads and no admin UI for managing them. A config file is simpler, version-controlled, and doesn't require a migration. When we hit 10+ spreads or need dynamic management, we'll move to a database.

2. **Why not just add an `if/else` for each spread type?** That works for three spreads but becomes unmanageable. The registry pattern means adding a new spread is a one-line config change with no code modifications to the assembly pipeline.

3. **Should viscosity affect jelly application too?** Potentially — a thin spread might make jelly slide around. Deferred for now. If we see jelly displacement issues during testing, we'll revisit.

## 🔮 Open Questions

1. **Inventory tracking**: The registry knows what spreads exist but not whether we have any in stock. If we add inventory later, should it live in the registry or in a separate system? Leaning toward separate — the registry is about *how* to handle a spread, inventory is about *whether* we can.

2. **Custom spread blends**: Some users might want peanut butter AND almond butter. The current model assumes one spread per sandwich. This would require rethinking the assembly pipeline's spread step as a loop. Not worth solving until users ask for it.
