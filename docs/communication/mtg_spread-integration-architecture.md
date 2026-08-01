# Spread Integration Architecture — Technical Alignment

**Date:** 2026-03-15
**Attendees:** Maya (eng), Tom (eng), Priya (product)
**Project:** pbj-machine
**Type:** Technical Alignment
**Related:** `cpf_assembly-pipeline.md`, `fsp_artisan-spread-selection.md`

# Summary

The team met to decide how the assembly controller should handle an expanding list of spread types beyond peanut butter. The team aligned on building a generic spread adapter layer, with peanut butter as the first migration target — proving the pattern on known-good behavior before new spreads ship. Two questions remain open: where temperature pre-handling logic should live, and whether the spread configuration should be stored in the database or baked into the application code.

# Index

**Decisions**
- ✅ **Spread adapter layer** — Build a layer that every spread plugs into the same way, so the controller never has to be changed when a new spread is added. Peanut butter is migrated onto it first to validate the pattern on known-good behavior before any new spread ships.

**Open Questions**
- ❓ **Temperature pre-handling location** — Should the adapter layer handle tempering a spread before use, or should the core controller own that? Affects how much the adapter needs to know about the machine's physical infrastructure. Owner: Maya. Due: before almond butter implementation begins.
- ❓ **Spread configuration storage** — Database vs. application code. Database means no code change needed to add a new spread; application code is simpler but requires a deployment for every new spread. Owner: Tom. Due: unscheduled, needed before this part of the system is built.

**Action Items**
- 👉 **Maya** — Draft the spread adapter design and migration plan for peanut butter — by March 22
- 👉 **Tom** — Write up configuration storage options with a recommendation — by March 20
- 👉 **Priya** — Confirm Q2 spread priority order with operations team — by March 18

# Conversation

- The meeting was prompted by a product goal: going into Q2, customers should be able to select spread types beyond peanut butter when building their sandwich — almond butter and sunflower butter are both queued. The problem is that peanut butter support is currently tangled throughout the part of the system that manages the sandwich-making sequence, and adding more spreads the same way will make that system unmaintainable. With four additional spreads already on the Q3 roadmap, the team needs to make an architectural decision now that can scale.
- Three options were considered for how to handle this:
    - Option 1 — Generic spread adapter layer: build a layer between the controller and the spreads. Every spread plugs in the same way, and the controller talks to that layer rather than to any spread directly. Adding a new spread means plugging into the layer, not changing the controller.
    - Option 2 — Per-spread conditional logic: continue adding new logic to the controller for each new spread. Faster in the short term, but the controller grows with every spread added.
    - Option 3 — Hybrid: leave peanut butter's existing logic in place, build the adapter layer for new spreads only, and migrate peanut butter to the adapter later.
- Option 2 was ruled out early. The controller is already hard to follow with one special-cased spread, and the trajectory from here gets worse with every spread added.
- Option 3 had initial appeal because it avoids touching working peanut butter logic. The problem is that peanut butter becomes a permanent exception that every engineer has to know about, and the adapter layer can't be fully validated until a known-good spread is on it — meaning the first real test of the pattern happens on a new, unproven spread rather than on something with a track record in production.
- Option 1 requires migrating peanut butter as the first implementation, which is more upfront work. The argument for it is that you validate the pattern on something that's already working before any new spread ships on top of it. If the adapter has a flaw, you find it during peanut butter migration, not during almond butter's launch week.
- ✅ **DECISION:** Build the generic spread adapter layer. Peanut butter is migrated onto it first — not a new spread — so the pattern is proven on known-good behavior before almond butter ships.
- Discussion then moved to what the adapter layer needs to know about each spread. Three things were identified: viscosity, which determines how much pressure the spreader applies; temperature requirements, which determine whether the spread needs any preparation before it can be applied; and a setup step the controller runs before the assembly sequence begins to handle any spread-specific preparation.
- Temperature pre-handling raised a design question. Some spreads need to be tempered to room temperature before they can be applied. Two options for where that logic lives: inside the adapter layer itself, which keeps the controller clean but means the adapter has to understand the machine's heating infrastructure; or in the core controller, which keeps all machine-level concerns in one place but couples the controller to spread-specific behavior.
- ❓ **OPEN:** Where does temperature pre-handling logic live — in the spread adapter layer or in the core assembly controller? The answer determines how much the adapter needs to know about the machine's physical infrastructure. Owner: Maya. Due: before almond butter implementation begins.
- The team also surfaced a question about how spread configurations are stored. The controller needs to look up how each spread behaves while a sandwich is being made — things like its viscosity and temperature requirements. Two options: store that information in the database, which means adding a new spread doesn't require a code change and can be done at any time; or bake it into the application code, which is simpler to set up but means adding a new spread requires a code change and a deployment.
- ❓ **OPEN:** Should spread configurations be stored in the database or baked into the application code? Owner: Tom. Due: unscheduled — needed before this part of the system is built, not before the adapter work begins.
- The team agreed on sequencing: build the adapter layer and migrate peanut butter first, get it reviewed and tested, then implement almond butter as the first new spread on the adapter. Having two real data points on the pattern before it's considered stable felt right.
- 👉 **ACTION:** Maya — draft the spread adapter interface and migration plan for peanut butter — by March 22.
- 👉 **ACTION:** Tom — write up the configuration storage options with a recommendation — by March 20.
- 👉 **ACTION:** Priya — confirm Q2 spread priority order with the operations team so engineering knows which spread follows almond butter — by March 18.
