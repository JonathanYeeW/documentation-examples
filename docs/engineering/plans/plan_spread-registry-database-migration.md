# Spread Registry Database Migration

**Created:** 2026-04-02
**Ticket:** PBJ-451
**Codebase:** `pbj-server`

# 📋 Summary

This plan moves the Spread Registry from a static config file to a database table without downtime. Phases 1–3 build the table and bring it level with the file while the file stays authoritative. Phases 4 and 5 move reads onto the table and remove the file.

# 🧭 Context

**Where this sits**

- The assembly pipeline reads the registry on every order, and it can't be paused.
- The registry is `spread-registry.ts`, a config file. Adding a spread means a code change and a deploy.
- The ops team can't add a spread without an engineer. The case for the move, and the admin UI it enables, is in PBJ-451.
- `SpreadStore`'s interface was settled in `fsp_spread-store.md`, so this plan names its methods.

**Terms**

- **Registry:** the set of spreads the machine can apply, with each one's label, category and handling settings.
- **Category:** `nut_butter`, `seed_butter` or `fruit_spread`. Cutover happens one category at a time.
- **Dual-write:** every registry write lands in both the table and the file, or in neither.
- **Cutover:** reads for a category move from the file to the table.

# 🏗️ Implementation

```
Phase 1: Table + data-access layer         reads still hit the file
Phase 2: Dual-write                        both stores stay in sync
Phase 3: Backfill + verification

    ~~~~~~~~ gate: all 34 spreads match row-for-row ~~~~~~~~

Phase 4: Read cutover behind a flag        nut_butter → seed_butter → fruit_spread
Phase 5: Delete the config file

  → phase 4 rolls back by flipping the flag; phase 5 is the point of no return
```

**Constraints**

- No downtime.
- Every registry access goes through `SpreadStore` from phase 1 onward. A direct import of the config file anywhere is a bug.
- The file stays authoritative until phase 4. Until then the table is a replica, and any disagreement is resolved in the file's favour.
- Rollback is a flag flip, not a deploy, for all of phase 4.
- Registry reads are on the hot path. `SpreadStore` caches, and invalidating the cache on write is part of phase 2, not an optimization for later.

## Phase 1: Table and Data-Access Layer

Create the `spreads` table and put `SpreadStore` in front of every registry read, with the config file still behind it. It goes first so every caller is on the new seam before anything moves.

- [ ] The `spreads` table exists, mirroring `SpreadConfig`
- [ ] `SpreadStore` serves `get`, `list` and `listByCategory` from the config file
- [ ] No module outside `SpreadStore` imports the config file
- [ ] Reads are cached, with a documented invalidation path

**Context**

- Columns: `id`, `label`, `refrigerate`, `stir`, `viscosity`, `category`
- Config file exports: `SPREAD_REGISTRY` and `getSpreadConfig`
- Known direct importers: `assembly-pipeline.ts`, `spread-selector.tsx`, `spread-handlers.ts`

## Phase 2: Dual-Write

Make every registry write land in both the table and the config file, so neither can drift. This is what makes the backfill safe to re-run and the cutover safe to reverse.

- [ ] `SpreadStore` offers `create`, `update` and `remove`
- [ ] A write lands in both stores or in neither
- [ ] Every successful write invalidates the cache
- [ ] Every write logs both targets and their results

**Context**

- The application has no write paths today. Spreads are added by editing the file and deploying
- The file write regenerates the exported object. It isn't a runtime append

## Phase 3: Backfill and Verification

Copy the 34 existing spreads into the table and prove the two stores agree. It's the gate before anything reads from the table.

- [ ] Every spread in the file is in the table, and a re-run adds nothing
- [ ] A field-by-field comparison of the two stores reports zero mismatches

**Context**

- Expected: 34 rows, made up of 18 `nut_butter`, 9 `seed_butter` and 7 `fruit_spread`
- The comparison covers every column, not just `id`, because a matching row count proves nothing

## Phase 4: Read Cutover

Move reads from the file to the table one category at a time, behind a flag, watching assembly errors between each. Going a category at a time means a bad cutover is limited to one category and reversible.

- [ ] Reads for each category can be switched between the file and the table without a deploy
- [ ] `nut_butter` on the table for 24 hours with no rise in assembly errors
- [ ] `seed_butter` on the table for 24 hours with no rise in assembly errors
- [ ] `fruit_spread` on the table for 24 hours with no rise in assembly errors
- [ ] All three on the table for a full week

**Context**

- Flag: `spread_registry_source`, values `file` or `table`, scoped per category
- Watch: assembly error rate, `UnknownSpreadError` count, registry read latency

## Phase 5: Delete the Config File

Remove the file, the dual-write path, the flag and the phase 3 tools, now that the table has been authoritative for a week. It's last because it can't be undone by a flag.

- [ ] Nothing in the codebase references the config file, the flag, or the backfill and verification tools
- [ ] Every `SpreadStore` write goes to the table only
- [ ] The assembly pipeline passes its integration suite

**Context**

- After this, rollback means restoring the file from git and running a backfill in reverse

# 📊 Current Status

| | Status | Notes |
|---|---|---|
| Phase 1 — table + data-access layer | ✅ Complete | Three direct importers replaced |
| Phase 2 — dual-write | ✅ Complete | No runtime write paths exercise it yet |
| Phase 3 — backfill + verification | 🟡 In progress | 34 rows inserted; verification reports 2 mismatches on `viscosity` |
| Phase 4 — read cutover | ⬜ Not started | Gated on a clean verification run |
| Phase 5 — delete the config file | ⬜ Not started | One week after phase 4 completes |
