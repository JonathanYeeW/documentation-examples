# PB&J Assembly Pipeline

**Created:** 2026-03-01
**Last Updated:** 2026-03-01
**Directory:** `sandwich-service`

Our assembly pipeline takes a user's sandwich order and produces a finished PB&J — from pulling ingredients out of storage to plating a cut sandwich. It handles bread prep, spread application, jelly layering, and final assembly as a single end-to-end flow. If you're working on any part of the sandwich system, this is the foundational process everything else builds on.

## Part 1: 🧈 Ingredient Preparation

Before anything gets assembled, the pipeline gathers and prepares all ingredients based on the user's selections. This is where we go from "user wants wheat bread with grape jelly" to "ingredients are staged and ready for assembly."

**Step 1:** **Resolve the order into concrete ingredients**
`order-resolver.ts`

When a user submits their sandwich order, it comes in as human-readable selections — "wheat," "grape," "creamy peanut butter." The order resolver maps these to specific inventory items with SKUs, storage locations, and handling instructions. Think of it as translating from the menu into kitchen language.

**Step 2:** **Pull ingredients from storage**
`inventory-service.ts`

Each ingredient lives in a specific storage zone. Bread is in the pantry at room temperature. Peanut butter is in the pantry. Jelly is refrigerated. The inventory service retrieves each item from its zone and confirms availability. If any ingredient is out of stock, the order fails here — we don't start assembling a sandwich we can't finish.

**Step 3:** **Stage ingredients at the prep station**
`prep-station.ts`

All ingredients arrive at the prep station in the order they'll be needed. Bread first (since it gets prepped first), then spread, then jelly. The staging step also runs quality checks — is the bread stale? Is the jelly expired? Items that fail quality checks get flagged and the user is notified before we proceed.

## Part 2: 🍞 Bread Handling

Bread is the foundation of the sandwich, and different bread types require different preparation. This part covers everything from unpackaging to getting two slices ready for spread application.

**Step 1:** **Select and slice bread**
`bread-handler.ts`

If the bread is pre-sliced (standard sandwich bread), we pull two slices from the bag. If it's a loaf (sourdough, artisan), we slice two pieces at 12mm thickness — thin enough to bite through easily but thick enough to hold the fillings without tearing. The slicer adjusts automatically based on bread type metadata from the order resolver.

**Step 2:** **Optional toasting**
`bread-handler.ts`

If the user selected toasted bread, the slices go through the toaster at a bread-type-specific setting. White bread toasts at level 3, wheat at level 4, sourdough at level 5. The key detail here: toasted bread needs a 30-second cool-down before spread application. If we spread peanut butter on hot bread, it melts and soaks in instead of sitting as a layer. The pipeline enforces this wait.

- **For toasted bread**: Toast → 30-second cool-down → proceed to spread
- **For untoasted bread**: Proceed directly to spread

**Step 3:** **Lay out slices for assembly**
`assembly-station.ts`

Both slices are placed face-up on the assembly surface. We designate Slice A (left) as the spread slice and Slice B (right) as the jelly slice. This assignment is fixed — spread always goes on the left slice. Consistency here matters because the cutting step later assumes this layout.

## Part 3: 🥜 Spread & Jelly Application

This is the core assembly step where the sandwich goes from "two slices of bread" to "a sandwich worth eating." Spread goes on one slice, jelly on the other, and both need proper coverage to avoid dry bites.

**Step 1:** **Apply peanut butter to Slice A**
`spread-applicator.ts`

The spreader dispenses 2 tablespoons of peanut butter onto the center of Slice A. It then spreads outward in a spiral pattern, covering the surface to within 5mm of the bread edge. We leave that margin so spread doesn't squeeze out the sides when the sandwich is pressed together.

The viscosity of the peanut butter determines spreader pressure. Creamy gets light pressure. Crunchy gets medium pressure with an extra pass to distribute chunks evenly. If the spreader detects resistance beyond the expected range (usually means the peanut butter has dried out), it flags a quality issue.

**Step 2:** **Apply jelly to Slice B**
`jelly-applicator.ts`

Jelly application follows the same spatial pattern as spread — center out, 5mm margin — but uses 1.5 tablespoons instead of 2. The ratio matters: too much jelly relative to peanut butter and the sandwich gets soggy. Too little and you mostly taste peanut butter.

Jelly is more liquid than peanut butter, so the applicator uses a different tool — a flat spatula rather than a knife. The spatula prevents jelly from soaking into the bread grain, which keeps the slice structurally sound through cutting and plating.

**Step 3:** **Validate coverage**
`quality-check.ts`

Before combining, a quick visual check confirms both slices have even coverage with no bare spots larger than 1cm². Bare spots mean dry bites, which is the most common user complaint. If coverage fails, the applicator does a touch-up pass on the sparse areas.

## Part 4: 🔪 Final Assembly & Plating

The last mile — combining the slices, cutting the sandwich, and getting it to the user.

**Step 1:** **Combine slices**
`assembly-station.ts`

Slice B (jelly side) flips onto Slice A (spread side), creating the sandwich. The flip is done in a single smooth motion to prevent jelly from sliding off. Once combined, light pressure is applied for 2 seconds to bond the fillings to both bread surfaces.

**Step 2:** **Cut the sandwich**
`cutting-service.ts`

The default cut is diagonal — corner to corner — producing two triangular halves. Diagonal cuts are the default because they create a longer exposed edge, which looks better on the plate and gives the user a natural grip point.

- **For diagonal cut (default)**: Single cut corner-to-corner
- **For rectangular cut**: Single horizontal cut through the center
- **For no cut**: Skip this step entirely

The cutting blade is cleaned between every sandwich to prevent cross-contamination between orders.

**Step 3:** **Plate and serve**
`plating-service.ts`

The halves are arranged on a plate with the cut edges facing outward (so the user can see the layers). A final quality photo is taken for the order record, and the sandwich is marked as complete in the order system. Total time from order submission to plated sandwich: approximately 3 minutes for standard orders.

---

*The PB&J assembly pipeline turns a user's sandwich selections into a finished, plated sandwich through four stages: ingredient prep, bread handling, spread and jelly application, and final assembly.*
