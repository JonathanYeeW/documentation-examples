# Spread Nutritional Backfill

**Created:** 2026-03-18
**Ticket:** PBJ-412
**Codebase:** `pbj-server`

## 📋 Summary

The Spread Registry ships with config for each spread — label, viscosity, handling flags — but no nutritional data. The health dashboard team needs per-spread calorie and allergen info to surface on the order screen. New spreads will get a `nutrition` block at creation time going forward, but the 34 existing registry entries need backfilling. Nutritional data lives inside the spread config object (not a separate table) since it's always read alongside the rest of the config and never queried independently. Backfill is idempotent — a spread is skipped if its config already has a `nutrition` block.

## 🏗️ Implementation

```
Phase 1: Build the atomic backfill endpoint and MCP tool
Phase 2: Build category-scoped bulk MCP tools
Phase 3: Run the full backfill across all 34 spreads
Phase 4: Post-backfill cleanup
```

### Phase 1: Atomic Backfill Endpoint and MCP Tool

A single server endpoint and matching MCP tool that accept an array of spread IDs, process each one, and return a per-spread status. This is the validation phase — we run a handful of real spreads through it, review the agent output, and confirm the behavior is correct before building any automation on top.

The endpoint accepts `{ ids: string[] }` and returns `{ id, status: "ok" | "skipped" | "error", message? }[]` — one entry per input ID. The handler fetches each spread config, checks whether a `nutrition` block already exists (skips if so), calls the `spreadNutritionAgent` with the spread's label and known ingredients, merges the returned block into the config, and saves.

The MCP tool wraps the same endpoint with human-readable output so the results can be reviewed directly from Claude Desktop before committing to scale.

### Phase 2: Category-Scoped Bulk MCP Tools

Three MCP tools — one per spread category — that scope the backfill automatically instead of requiring manual ID lists. Each tool fetches all spreads in its category that don't yet have a `nutrition` block, then drives the Phase 1 endpoint in batches of 10 with a short delay between batches.

The three categories are `nut_butter` (18 spreads), `seed_butter` (9 spreads), and `fruit_spread` (7 spreads). These live entirely in the MCP server — no new server endpoints. Phase 1 does all the actual work; Phase 2 is orchestration on top of proven infrastructure.

Build order within Phase 2: `nut_butter` first for maximum coverage, validate the output, then `seed_butter`, then `fruit_spread`. The `fruit_spread` category has an open question that needs resolving before it can run — see below.

### Phase 3: Full Backfill Run

Run the three Phase 2 tools in sequence. Confirm per-category output and check for errors. Re-run any failed IDs using the Phase 1 tool directly.

### Phase 4: Post-Backfill Cleanup

Once all 34 spreads have a `nutrition` block confirmed, remove the legacy `calories_estimate` field that previously existed on some configs. This is a data migration — fetch every spread config, delete `calories_estimate` if present, and save.

## 📊 Current Status

| | Status | Notes |
|---|---|---|
| Phase 1 — atomic endpoint + MCP tool | ✅ Complete | Validated on 5 spreads |
| Phase 2 — category bulk tools | ⬜ Not started | Build `nut_butter` first |
| Phase 3 — full backfill run | ⬜ Not started | After Phase 2 validated |
| Phase 4 — post-backfill cleanup | ⬜ Not started | After Phase 3 complete |

## ❓ Open Questions

1. **`fruit_spread` ingredient sourcing** — The nutrition agent derives calorie estimates from ingredient lists. Fruit spreads only have a flavor label in the registry, not a stable ingredient list. Need to decide: hardcode ingredient lists for the 7 fruit spreads, or add an `ingredients` field to the registry before running Phase 2 for that category.

2. **Allergen display format** — The agent returns allergens as a string array (`["peanuts", "tree nuts"]`). The health dashboard team hasn't confirmed whether they want the raw array or a pre-formatted string. Confirm before Phase 4 to avoid having to re-run the backfill.
