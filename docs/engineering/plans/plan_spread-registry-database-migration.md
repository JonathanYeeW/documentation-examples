# Spread Registry Database Migration

**Created:** 2026-04-02
**Ticket:** PBJ-451
**Codebase:** `pbj-server`

# 📋 Summary

This plan describes how to move the Spread Registry from a static config file to a database table without downtime. It covers five phases: creating the table, dual-writing to both stores, backfilling the existing spreads, cutting reads over behind a flag, and deleting the config file.

# 🧭 Approach

The registry has outgrown a file. Every new spread needs a code change and a deploy, and the ops team can't add one without an engineer. Moving it to a table means an admin UI becomes possible, but the assembly pipeline reads the registry on every order, so the swap has to happen underneath live traffic.

1. **Create the table and a data-access layer that reads from the file.** The layer ships first with the file still behind it, so every caller is already going through the new seam before anything moves.
2. **Dual-write, so the table and the file never disagree.** Any write goes to both. This is what makes the backfill safe to run more than once and the cutover safe to reverse.
3. **Backfill the 34 existing spreads and verify the two stores match.** A row-by-row comparison is the gate — if the table and the file disagree on any spread, the cutover doesn't happen.
4. **Flip reads to the table behind a flag**, one category at a time, watching assembly error rates between each.
5. **Delete the file and the dual-write path** once reads have been on the table for a full week with no rollback.

# 🏗️ Implementation

```
Phase 1: Table + data-access layer         reads still hit the file
Phase 2: Dual-write                        both stores stay in sync
Phase 3: Backfill + verification
Phase 4: Read cutover behind a flag        nut_butter → seed_butter → fruit_spread
Phase 5: Delete the config file

    ~~~~~~~~ gate: all 34 spreads match row-for-row ~~~~~~~~

  → phase 4 rolls back by flipping the flag; phase 5 is the point of no return
```

**Constraints**

- No downtime. The assembly pipeline reads the registry on every order and cannot be paused.
- Every registry access goes through the data-access layer from phase 1 onward. A direct import of the config file anywhere is a bug.
- The file stays authoritative until phase 4. Until then the table is a replica, and any disagreement is resolved in the file's favour.
- Rollback is a flag flip, not a deploy, for all of phase 4.
- Registry reads are on the hot path. The data-access layer caches, and cache invalidation on write is part of phase 2, not an optimization for later.

## Phase 1: Table and Data-Access Layer

Create the `spreads` table and put a data-access layer in front of every registry read, with the config file still behind it.

- [ ] Write the migration for the `spreads` table mirroring `SpreadConfig`
- [ ] Build `SpreadStore` with `get`, `list`, and `listByCategory`
- [ ] Point `SpreadStore` at the config file
- [ ] Replace every direct `SPREAD_REGISTRY` import with a `SpreadStore` call
- [ ] Add an in-memory cache with a documented invalidation path
- [ ] Confirm no module outside `SpreadStore` imports the config file

**Context**

- Table: `spreads`, columns mirroring `SpreadConfig` — `id`, `label`, `refrigerate`, `stir`, `viscosity`, `category`
- Existing config: `spread-registry.ts`, exporting `SPREAD_REGISTRY` and `getSpreadConfig`
- Known direct importers: `assembly-pipeline.ts`, `spread-selector.tsx`, `spread-handlers.ts`

## Phase 2: Dual-Write

Make every registry write land in both the table and the config file, so neither can drift from the other.

- [ ] Add write methods to `SpreadStore` — `create`, `update`, `remove`
- [ ] Write to the table first, then the file, in one transaction where possible
- [ ] Fail the whole write if either store rejects it
- [ ] Invalidate the cache on every successful write
- [ ] Add a structured log line naming both write targets and their results

**Context**

- Write paths today: none in the application. Spreads are added by editing the file and deploying
- The file write is a code-generation step, not a runtime append — it rewrites the exported object

## Phase 3: Backfill and Verification

Copy the 34 existing spreads into the table and prove the two stores agree before anything reads from the new one.

- [ ] Write `backfill-spreads` to read the file and insert each spread
- [ ] Skip any spread already present in the table so the job can re-run
- [ ] Write `verify-spreads` to compare the two stores field by field
- [ ] Run the backfill and then the verification
- [ ] Re-run both until verification reports zero mismatches

**Context**

- Expected: 34 rows — 18 `nut_butter`, 9 `seed_butter`, 7 `fruit_spread`
- Expected: zero field-level mismatches across all 34
- Verification compares every column, not just `id`. A matching row count proves nothing

## Phase 4: Read Cutover

Flip reads from the file to the table one category at a time, behind a flag, watching assembly errors between each.

- [ ] Add the `spread_registry_source` flag, defaulting to `file`
- [ ] Have `SpreadStore` branch on the flag per category
- [ ] Cut `nut_butter` over and watch assembly error rates for 24 hours
- [ ] Cut `seed_butter` over and watch for 24 hours
- [ ] Cut `fruit_spread` over and watch for 24 hours
- [ ] Leave all three on the table for a full week before phase 5

**Context**

- Flag: `spread_registry_source`, values `file` or `table`, scoped per category
- Rollback: flip the flag back. No deploy, no data change
- Watch: assembly error rate, `UnknownSpreadError` count, registry read latency

## Phase 5: Delete the Config File

Remove the file, the dual-write path, and the flag now that the table has been authoritative for a week.

- [ ] Delete `spread-registry.ts`
- [ ] Remove the file-write half of every `SpreadStore` write method
- [ ] Remove the `spread_registry_source` flag and its branch
- [ ] Delete `backfill-spreads` and `verify-spreads`
- [ ] Confirm the assembly pipeline still passes its integration suite

**Context**

- This is the point of no return. After it, rollback means restoring the file from git and re-running a backfill in reverse

# 📊 Current Status

| | Status | Notes |
|---|---|---|
| Phase 1 — table + data-access layer | ✅ Complete | Three direct importers replaced |
| Phase 2 — dual-write | ✅ Complete | No runtime write paths exercise it yet |
| Phase 3 — backfill + verification | 🟡 In progress | 34 rows inserted; verification reports 2 mismatches on `viscosity` |
| Phase 4 — read cutover | ⬜ Not started | Gated on a clean verification run |
| Phase 5 — delete the config file | ⬜ Not started | One week after phase 4 completes |
