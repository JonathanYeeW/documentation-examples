# Spread Nutritional Backfill

**Created:** 2026-03-18
**Ticket:** PBJ-412
**Codebase:** `pbj-server`

# 📋 Summary

This plan backfills nutritional data onto the 34 spreads already in the Spread Registry, and then removes the legacy field it replaces. Phases 1 and 2 build the tools, phase 3 runs them, and phase 4 cleans up. Phases 2–4 get detail as the phase before them finishes, since each depends on what the earlier run shows.

# 🧭 Context

**Where this sits**

- The health dashboard shows calories and allergens for each spread.
- New spreads get nutritional data when they're created. The 34 that predate that have none.
- Some older spreads carry a `calories_estimate` field. The dashboard reads it today.
- An agent can derive calories and allergens from a spread's label and ingredient list. PBJ-412 records why an agent was chosen over manual entry.

**Terms**

- **Nutrition block:** the calories and allergens stored inside a spread's config object.
- **Atomic backfill:** backfilling an explicit list of spread IDs, returning one result per ID.
- **Bulk tool:** an MCP tool that finds every spread in one category without a nutrition block and runs the atomic backfill over them in batches.

# 🏗️ Implementation

```
Phase 1: Atomic backfill endpoint + MCP tool      validation gate
Phase 2: Category-scoped bulk MCP tools           nut_butter → seed_butter → fruit_spread
Phase 3: Full backfill run                        all 34 spreads
Phase 4: Remove calories_estimate

    ~~~~~~~~ gate: phase 1 output reviewed by hand ~~~~~~~~
```

**Constraints**

- The nutrition block lives inside the spread config object, not a separate table. It's always read with the rest of the config and never queried on its own.
- The backfill is idempotent. A spread that already has a nutrition block is skipped.
- Phase 2 adds no server endpoints. The bulk tools live in the MCP server and drive the phase 1 endpoint.

## Phase 1: Atomic Backfill Endpoint and MCP Tool

Build the endpoint that backfills an explicit list of spread IDs, and an MCP tool that makes its output reviewable by hand. Everything after is orchestration on top of it, so its output is reviewed before any automation exists.

- [x] Given a list of IDs, the endpoint returns one status per ID
- [x] A spread with a nutrition block is skipped
- [x] A backfilled spread's config carries the agent's nutrition block
- [x] The MCP tool shows the output readably
- [x] Five real spreads run through it, and their output is reviewed by hand

**Context**

- Endpoint response shape: `{ id, status: "ok" | "skipped" | "error", message? }[]`
- Agent: `spreadNutritionAgent`, which takes a label and an ingredient list

## Phase 2: Category-Scoped Bulk MCP Tools

Build one bulk tool per spread category, starting with the largest. Each one is validated before the next is built, so a problem found in `nut_butter` isn't copied twice.

- [ ] The `nut_butter` tool backfills its 18 spreads, and its output is validated
- [ ] The `seed_butter` tool backfills its 9
- [ ] The `fruit_spread` tool backfills its 7

**Context**

- Categories: `nut_butter` (18), `seed_butter` (9), `fruit_spread` (7)
- `fruit_spread` is blocked. The registry holds only a flavour label for these, and the agent needs an ingredient list. The options are to hardcode lists for the seven, or add an `ingredients` field to the registry
- Starting ideas, to test here rather than take as decided: batches of 10, with a short delay between batches

## Phase 3: Full Backfill Run

Run the three bulk tools and confirm every spread has a nutrition block. Failures are retried one at a time through the phase 1 tool.

- [ ] All 34 spreads have a nutrition block, with zero errors

**Context**

- Expected: 34 spreads with a nutrition block. That's 18 `nut_butter`, 9 `seed_butter` and 7 `fruit_spread`

## Phase 4: Remove `calories_estimate`

Delete the legacy field the nutrition block replaces. It's last because the dashboard reads the field until every spread has real data.

- [ ] No spread config has `calories_estimate`
- [ ] The health dashboard reads only from the nutrition block

**Context**

- `calories_estimate` is present on some configs, not all
- The allergen format is unconfirmed. The agent returns a string array, and the health dashboard team hasn't said whether they want that or a pre-formatted string. It has to be settled before this phase, or the backfill has to re-run

# 📊 Current Status

| | Status | Notes |
|---|---|---|
| Phase 1 — atomic endpoint + MCP tool | ✅ Complete | Validated on 5 spreads |
| Phase 2 — category bulk tools | ⬜ Not started | `fruit_spread` blocked on sourcing ingredient lists |
| Phase 3 — full backfill run | ⬜ Not started | Gated on phase 2's validated tools |
| Phase 4 — remove `calories_estimate` | ⬜ Not started | Gated on phase 3, and on the allergen format |
