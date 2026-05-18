# PB&J Assembly Pipeline

**Created:** 2026-03-01
**Last Updated:** 2026-03-11
**Directory:** `sandwich-service`

Our assembly pipeline takes a user's sandwich order and produces a finished PB&J — from pulling ingredients out of storage to plating a cut sandwich. It handles bread prep, spread application, jelly layering, and final assembly as a single end-to-end flow.

Read this when you need to understand how the full pipeline is sequenced, why steps happen in the order they do, or what constraints downstream steps depend on. If you're modifying any part of the sandwich system, understanding this flow first will tell you what you might break.

## Part 1: 🧈 Ingredient Preparation

Before anything gets assembled, the pipeline gathers and prepares all ingredients based on the user's selections. This is where we go from "user wants wheat bread with grape jelly" to "ingredients are staged and ready for assembly."

**Step 1:** **Resolve the order into concrete ingredients**
`order-resolver.ts`

When a user submits their sandwich order, it comes in as human-readable selections — "wheat," "grape," "creamy peanut butter." The order resolver maps these to specific inventory items with SKUs, storage locations, and handling instructions. Every downstream step depends on the resolved output from this step — if the mapping is wrong or incomplete, the error propagates silently through prep and surfaces as a bad sandwich, not a failed order.

**Step 2:** **Pull ingredients from storage**
`inventory-service.ts`

Each ingredient lives in a specific storage zone. Bread is in the pantry at room temperature. Peanut butter is in the pantry. Jelly is refrigerated. The inventory service retrieves each item from its zone and confirms availability. If any ingredient is out of stock, the order fails here — we don't start assembling a sandwich we can't finish. Failing fast at this step is intentional: assembly is not easily reversible once it starts.

**Step 3:** **Stage ingredients at the prep station**
`prep-station.ts`

All ingredients arrive at the prep station in the order they'll be used: bread first, then spread, then jelly. The staging step also runs quality checks — is the bread stale? Is the jelly expired? Items that fail are flagged and the user is notified before we proceed. Quality checks happen here, not mid-assembly, because catching a bad ingredient after the spread is already applied wastes both the ingredient and the assembly time.

## Part 2: 🍞 Bread Handling

Bread is the foundation of the sandwich, and different bread types require different preparation. This part covers everything from unpackaging to getting two slices ready for spread application.

**Step 1:** **Select and slice bread**
`bread-handler.ts`

If the bread is pre-sliced (standard sandwich bread), we pull two slices from the bag. If it's a loaf (sourdough, artisan), we slice two pieces at 12mm thickness — thin enough to bite through easily but thick enough to hold the fillings without tearing. The 12mm spec exists because thinner slices tear during cutting and thicker ones make the sandwich hard to bite through. The slicer adjusts automatically based on bread type metadata from the order resolver.

**Step 2:** **Optional toasting**
`bread-handler.ts`

If the user selected toasted bread, the slices go through the toaster at a bread-type-specific setting. White bread toasts at level 3, wheat at level 4, sourdough at level 5. Toasted bread requires a 30-second cool-down before spread application — peanut butter applied to hot bread melts and soaks in rather than sitting as a layer, which affects both texture and coverage validation in Part 3. The pipeline enforces this wait; skipping it causes coverage checks to pass incorrectly because the spread looks even on a hot surface but redistributes as it cools.

- **For toasted bread**: Toast → 30-second cool-down → proceed to spread
- **For untoasted bread**: Proceed directly to spread

**Step 3:** **Lay out slices for assembly**
`assembly-station.ts`

Both slices are placed face-up on the assembly surface. Slice A (left) is always the spread slice; Slice B (right) is always the jelly slice. This assignment is fixed because the cutting step in Part 4 assumes a specific orientation to produce clean halves. Changing which slice gets which filling without updating the cutting logic will produce sandwiches with uneven filling distribution at the cut edge.

## Part 3: 🥜 Spread & Jelly Application

Spread goes on Slice A, jelly on Slice B, and both need even coverage before the slices are combined. The order within this part matters — spread first, then jelly, then validate — because the validation step checks both slices together.

**Step 1:** **Apply peanut butter to Slice A**
`spread-applicator.ts`

The spreader dispenses 2 tablespoons of peanut butter onto the center of Slice A, then spreads outward in a spiral pattern to within 5mm of the bread edge. The 5mm margin prevents spread from squeezing out when the slices are pressed together in Part 4. Reducing the margin causes bleed; increasing it causes dry edges.

Spreader pressure varies by type: creamy uses light pressure, crunchy uses medium with an extra distribution pass to move chunks evenly. If the spreader detects resistance beyond the expected range for the selected type, it flags a quality issue — this usually means the peanut butter has dried out and won't spread correctly regardless of pressure.

**Step 2:** **Apply jelly to Slice B**
`jelly-applicator.ts`

Jelly follows the same spatial pattern — center out, 5mm margin — but uses 1.5 tablespoons instead of 2. The 3:2 spread-to-jelly ratio is intentional: more jelly than that makes the sandwich soggy; less means the spread flavor dominates. Jelly uses a flat spatula rather than a knife because a knife drags into the bread grain and creates channels that accelerate soaking. The spatula keeps the slice structurally intact through cutting and plating.

**Step 3:** **Validate coverage**
`quality-check.ts`

Before combining, a visual check confirms both slices have no bare spots larger than 1cm². Bare spots are the leading cause of user complaints. If coverage fails, the applicator does a touch-up pass on the sparse areas. This check runs after both slices are applied rather than after each individually because coverage on one slice can be affected by the time elapsed before combination — validating too early on the spread slice can miss drying at the edges.

## Part 4: 🔪 Final Assembly & Plating

The last mile — combining the slices, cutting the sandwich, and completing the order.

**Step 1:** **Combine slices**
`assembly-station.ts`

Slice B (jelly side) flips onto Slice A (spread side). The flip happens in a single smooth motion — hesitation mid-flip allows jelly to slide toward the edge before the slices meet. Once combined, light pressure is applied for 2 seconds to bond the fillings to both bread surfaces. Over-pressing compresses the bread and causes filling bleed past the 5mm margin.

**Step 2:** **Cut the sandwich**
`cutting-service.ts`

The default cut is diagonal — corner to corner — producing two triangular halves. Diagonal is the default because it creates a longer exposed edge, which surfaces more of the filling visually and gives the user a natural grip point. The cutting blade is cleaned between every sandwich; residue from one order's spread or jelly affects the cut quality and appearance of the next.

- **For diagonal cut (default)**: Single cut corner-to-corner
- **For rectangular cut**: Single horizontal cut through the center
- **For no cut**: Skip this step entirely

**Step 3:** **Plate and serve**
`plating-service.ts`

The halves are arranged with cut edges facing outward so the user can see the layers. A final quality photo is taken for the order record before the sandwich is marked complete. Total time from order submission to plated sandwich: approximately 3 minutes for standard orders. Toasted orders add 30–60 seconds depending on bread type.

---

*The PB&J assembly pipeline runs in four sequential stages — ingredient prep, bread handling, spread and jelly application, and final assembly — each with upstream dependencies that constrain what can change safely downstream.*
