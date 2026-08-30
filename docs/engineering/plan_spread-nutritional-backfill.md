# Spread Nutritional Backfill

**Created:** 2026-03-18
**Ticket:** PBJ-412
**Codebase:** `pbj-server`

# 📋 Summary

This plan describes how to backfill nutritional data onto the 34 spreads already in the Spread Registry. It covers four phases: building the atomic backfill endpoint, wrapping it in category-scoped bulk tools, running the full backfill, and removing the legacy field it replaces.

# 🧭 Approach

The backfill is a server endpoint plus a set of MCP tools that drive it. An agent derives calorie and allergen data from each spread's label and ingredient list, and the result is merged into the spread's existing config object.

1. **Build one endpoint that backfills an explicit list of spread IDs, and an MCP tool that wraps it.** Everything else is orchestration on top of this, so it has to be right first. A handful of real spreads run through it and the output is reviewed by hand before any automation exists.
2. **Wrap the endpoint in one bulk tool per spread category.** Each tool scopes the ID list automatically instead of requiring it by hand, then drives the endpoint in batches.
3. **Run the three bulk tools in sequence and re-run any failures individually.** The atomic tool from phase 1 is the retry mechanism.
4. **Remove the legacy `calories_estimate` field** once every spread has a confirmed `nutrition` block.

# 🏗️ Implementation

```
Phase 1: Atomic backfill endpoint + MCP tool      validation gate
Phase 2: Category-scoped bulk MCP tools           nut_butter → seed_butter → fruit_spread
Phase 3: Full backfill run                        all 34 spreads
Phase 4: Remove calories_estimate

    ~~~~~~~~ gate: phase 1 output reviewed by hand ~~~~~~~~
```

**Constraints**

- Nutritional data lives inside the spread config object, not a separate table. It's always read alongside the rest of the config and never queried independently.
- The backfill is idempotent. A spread with an existing `nutrition` block is skipped, so a re-run costs nothing.
- New spreads get a `nutrition` block at creation time. This backfill exists only for the 34 that predate that.
- Phase 2 adds no server endpoints. All three bulk tools live in the MCP server and drive the phase 1 endpoint.

## Phase 1: Atomic Backfill Endpoint and MCP Tool

Build the endpoint that backfills an explicit list of spread IDs, and the MCP tool that makes its output reviewable by hand.

- [x] Accept `{ ids: string[] }` and return one status entry per input ID
- [x] Fetch each spread config and skip it if a `nutrition` block already exists
- [x] Call `spreadNutritionAgent` with the spread's label and known ingredients
- [x] Merge the returned block into the config and save
- [x] Wrap the endpoint in an MCP tool with human-readable output
- [x] Run five real spreads through it and review the agent output

**Context**

- Endpoint response shape: `{ id, status: "ok" | "skipped" | "error", message? }[]`
- Agent: `spreadNutritionAgent`, takes label and ingredient list

## Phase 2: Category-Scoped Bulk MCP Tools

Build one MCP tool per spread category that scopes the ID list automatically and drives the phase 1 endpoint in batches.

- [ ] Build the `nut_butter` tool — 18 spreads
- [ ] Validate its output before building the others
- [ ] Build the `seed_butter` tool — 9 spreads
- [ ] Build the `fruit_spread` tool — 7 spreads
- [ ] Have each tool fetch spreads in its category lacking a `nutrition` block
- [ ] Drive the phase 1 endpoint in batches of 10 with a short delay between batches

**Context**

- Categories: `nut_butter` (18), `seed_butter` (9), `fruit_spread` (7)
- Build order: `nut_butter` first for maximum coverage
- `fruit_spread` is blocked — the registry holds only a flavor label for these, and the agent needs an ingredient list. Either hardcode lists for the seven, or add an `ingredients` field to the registry

## Phase 3: Full Backfill Run

Run the three bulk tools in sequence and confirm all 34 spreads come back with a `nutrition` block.

- [ ] Run the `nut_butter` tool and check its output
- [ ] Run the `seed_butter` tool and check its output
- [ ] Run the `fruit_spread` tool and check its output
- [ ] Re-run any failed IDs through the phase 1 tool directly

**Context**

- Expected: 34 spreads with a `nutrition` block, zero errors

## Phase 4: Remove `calories_estimate`

Delete the legacy field the `nutrition` block replaces, now that every spread carries real data.

- [ ] Confirm all 34 spreads have a `nutrition` block
- [ ] Fetch every spread config, delete `calories_estimate` if present, and save
- [ ] Confirm the health dashboard reads only from `nutrition`

**Context**

- Legacy field: `calories_estimate`, present on some configs but not all
- Allergen format is unconfirmed — the agent returns a string array and the health dashboard team hasn't said whether they want that or a pre-formatted string. Settle it before this phase, or the backfill has to re-run

# 📊 Current Status

| | Status | Notes |
|---|---|---|
| Phase 1 — atomic endpoint + MCP tool | ✅ Complete | Validated on 5 spreads |
| Phase 2 — category bulk tools | ⬜ Not started | `fruit_spread` blocked on ingredient sourcing |
| Phase 3 — full backfill run | ⬜ Not started | After phase 2 validated |
| Phase 4 — remove `calories_estimate` | ⬜ Not started | After phase 3 complete |
