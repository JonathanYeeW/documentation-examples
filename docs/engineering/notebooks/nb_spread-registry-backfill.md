# Notebook — Spread Registry Backfill

**Ticket:** PBJ-451
**Started:** 2026-04-19
**Status:** ✅ Complete — 2026-04-21
**Purpose:** Fill the `spreads` table from the config file and move every registry read onto it, underneath live traffic. Done when all 34 spreads match the file row for row and all three categories are reading from the table with the assembly error rate unchanged.

## 📊 Resource Register

Current state of what this operation created. Overwritten as it changes.

| Resource | Identifier | Notes |
|---|---|---|
| Table rows | `spreads` | 34 rows, matching `spread-registry.ts` row for row. Verified 2026-04-20 |
| Backfill script | `scripts/backfill-spreads.ts` | Idempotent on `id`. Refuses to run when dual-write is off |
| Flag | `registry.read_from_table.nut_butter` | ✅ On, 2026-04-21 09:14 |
| Flag | `registry.read_from_table.seed_butter` | ✅ On, 2026-04-21 10:02 |
| Flag | `registry.read_from_table.fruit_spread` | ✅ On, 2026-04-21 10:31 |
| Config file | `src/config/spread-registry.ts` | Still on disk and still dual-written. Deleted in phase 5, which is not this operation |

## 🔭 Context

Preconditions inherited from phases 1 and 2 of `plan_spread-registry-database-migration.md`. Each one constrains a command below.

- **The assembly pipeline reads the registry on every order and cannot be paused.** Every step here runs under live traffic, so nothing may hold a lock on `spreads` longer than a single row write.
- **The file is authoritative until the read cutover.** The table is a replica, and any disagreement between them is resolved in the file's favour — that is what makes the backfill safe to run more than once.
- **Dual-write is live.** Every registry write already goes to both stores, so the table drifts from the file only for spreads that existed before dual-write shipped.
- **`SpreadStore` caches reads in memory with a 60-second TTL.** A verification query issued through the application reads the cache, not the table. Verification has to go to the database directly.
- **The registry has three categories** — `nut_butter`, `seed_butter`, `fruit_spread` — and the cutover flag is per category.

## 📇 Entry Index

| # | | Title |
|---|---|---|
| 001 | 🔍 | What the table holds before anything runs |
| 002 | ⚖️ | Rehearse against a restored snapshot, not against staging |
| 003 | 🔧 | First rehearsal run — failed on a duplicate id |
| 004 | 🔍 | Where the duplicate came from |
| 005 | ⚖️ | Resolve duplicates in the file's favour, and fail on anything else |
| 006 | 📄 | The backfill script |
| 007 | 🔧 | Second rehearsal run, clean |
| 008 | 🔧 | Backfill production |
| 009 | 🔍 | Row-for-row verification — the gate |
| 010 | 🔍 | One mismatch: viscosity stored at the wrong precision |
| 011 | 🔧 | Correct the column type and re-run the single row |
| 012 | 🔧 | Cut `nut_butter` over |
| 013 | 🔍 | Assembly error rate through the first hour |
| 014 | 🔧 | Cut the remaining two categories |

## 🧪 Phase 1 — Rehearsal

**2026-04-19.** Prove the backfill against a copy of production before it touches production. Nothing in this phase runs against the live database.

### 001 🔍 What the table holds before anything runs

**Intent.** Establish the starting state. Dual-write shipped four days ago, so the table is not necessarily empty — any spread created or edited since then is already in it, and the backfill has to be correct in the presence of those rows rather than assuming a clean table.

**Command.**

```sql
SELECT count(*) AS rows, min(created_at) AS first, max(created_at) AS last FROM spreads;
SELECT id, label, category FROM spreads ORDER BY created_at;
```

**Output.**

```
 rows |          first           |          last
------+--------------------------+--------------------------
    2 | 2026-04-16 11:02:41+00   | 2026-04-17 15:48:09+00

 id                | label              | category
-------------------+--------------------+--------------
 hazelnut_cocoa    | Hazelnut Cocoa     | nut_butter
 apricot_preserve  | Apricot Preserve   | fruit_spread
```

**Settled.** The table is not empty. Two spreads were added through the admin path after dual-write shipped and exist only because of it.

Both also exist in the config file, because dual-write writes to both stores — so the backfill will encounter them and must not fail on them or duplicate them. **The backfill has to be idempotent on `id`, and that is now a requirement rather than a nicety.**

The file holds 34 spreads. The table holds 2 of them. The gap is 32.

---

### 002 ⚖️ Rehearse against a restored snapshot, not against staging

**Intent.** Decide what the backfill is rehearsed against before writing it.

**Options.** Run it against the staging database, which is always available and costs nothing to reset. Or restore last night's production snapshot into a scratch database and run against that.

**Decision.** The restored snapshot.

**Why.** Staging's `spreads` table was seeded by a fixture, and entry 001 is the reason that matters — the two rows that exist in production came from four days of real admin writes, and staging has no equivalent. A rehearsal against staging would prove the backfill works on an empty table, which is the one case production is known not to be in. The snapshot costs twenty minutes to restore and reproduces the exact condition the script has to survive.

**Consequence.** The rehearsal is not repeatable for free — each clean re-run needs the snapshot restored again. That is the price of rehearsing against the real shape, and it is why entry 003's failure is worth a full entry rather than a retry.

---

### 003 🔧 First rehearsal run — failed on a duplicate id

**Intent.** Run the backfill against the restored snapshot and see what it does.

**Command.**

```
pnpm backfill-spreads --database pbj_rehearsal_0419 --dry-run=false
```

**Output.**

```
Reading spread-registry.ts ... 34 spreads
Writing to spreads ...

  inserted  hazelnut_cocoa       ERROR
  duplicate key value violates unique constraint "spreads_pkey"
  DETAIL:  Key (id)=(hazelnut_cocoa) already exists.

Aborted after 0 writes. No partial state — the run is inside one transaction.
```

**Settled.** The run failed on the first spread that already existed, which is exactly the condition entry 001 predicted and the script did not handle.

**The transaction is the reason this is recoverable.** The whole backfill runs inside one, so a failure at row 1 or row 33 leaves the table exactly as it was. Confirmed by re-reading the count: still 2.

**Not yet known** is whether `hazelnut_cocoa` in the file and `hazelnut_cocoa` in the table hold the *same* values. A plain skip-if-exists would paper over a real disagreement between the two stores. That is 004.

---

### 004 🔍 Where the duplicate came from

**Intent.** Establish whether the two pre-existing rows agree with the file, before deciding what the script does about them. If they agree, the duplicate is noise. If they disagree, the duplicate is a data problem and skipping it silently would hide it.

**Command.**

```sql
SELECT id, label, refrigerate, stir, viscosity, category
FROM spreads WHERE id IN ('hazelnut_cocoa','apricot_preserve');
```

**Output.**

```
 id               | label             | refrigerate | stir  | viscosity | category
------------------+-------------------+-------------+-------+-----------+--------------
 hazelnut_cocoa   | Hazelnut Cocoa    | f           | t     |      0.62 | nut_butter
 apricot_preserve | Apricot Preserve  | t           | f     |      0.31 | fruit_spread
```

Against the file:

```
hazelnut_cocoa    { label: "Hazelnut Cocoa",   refrigerate: false, stir: true,  viscosity: 0.62, category: "nut_butter" }
apricot_preserve  { label: "Apricot Preserve", refrigerate: true,  stir: false, viscosity: 0.31, category: "fruit_spread" }
```

**Settled.** Both rows match the file on every column. The duplicates are dual-write working correctly, not drift.

**This is the answer that makes a skip safe** — but only for these two, and only today. A skip that assumes agreement would be wrong the first time a row *does* disagree, and there is no reason to expect that case never happens. The script needs to distinguish the two.

---

### 005 ⚖️ Resolve duplicates in the file's favour, and fail on anything else

**Intent.** Settle what the backfill does when a row already exists.

**Options.** Skip existing rows. Overwrite them from the file. Or compare, overwrite where they match trivially, and abort where they disagree.

**Decision.** Overwrite from the file, and abort the whole run if the overwrite would change a value.

**Why.** The plan's own constraint is that the file is authoritative until the read cutover, so writing the file's value is never wrong. What is not acceptable is doing it *silently*: a disagreement between the two stores means dual-write has a bug, and the backfill is the one moment that bug is visible. Overwriting quietly would repair the symptom and destroy the evidence in the same statement.

Aborting is affordable because of 003 — the run is one transaction, so an abort costs nothing but the re-run.

**Consequence.** The script cannot be run unattended, and that is deliberate. It also means the gate in 009 is a second, independent check rather than a restatement of this one: this compares the file to the table at write time, and 009 compares them again afterward, reading the table directly rather than trusting what the script reported.

---

### 006 📄 The backfill script

**Intent.** Apply 005. One script, run by hand, safe to run more than once.

**Path.** `scripts/backfill-spreads.ts`

**Shape.**

| Choice | Why |
|---|---|
| One transaction around the whole run | A failure at any row leaves the table exactly as it was. Proven in 003 before it was relied on |
| `INSERT ... ON CONFLICT (id) DO UPDATE`, with a pre-read comparison that aborts on any changed value | The write is idempotent; the comparison is what turns a silent repair into a visible failure |
| Refuses to start when the dual-write flag is off | With dual-write off, the table starts drifting the moment the script finishes, and a verified table that is already stale is worse than no table |
| Writes row by row rather than in one bulk statement | The assembly pipeline reads this table on every order. A bulk write holds a lock long enough to be visible on the hot path |
| `--dry-run` defaults to true | The destructive form has to be asked for |
| Prints a per-row line and a final count | The count is what entry 009's gate is checked against |

**Settled.** The script exists and encodes the decision in 005. It is not wired into the deploy and is not scheduled — it is run by hand, once per database.

---

### 007 🔧 Second rehearsal run, clean

**Intent.** Restore the snapshot again and run the script as written.

**Command.**

```
pnpm restore-snapshot --into pbj_rehearsal_0419
pnpm backfill-spreads --database pbj_rehearsal_0419 --dry-run=false
```

**Output.**

```
Reading spread-registry.ts ... 34 spreads
Dual-write flag: ON
Writing to spreads ...

  matched   hazelnut_cocoa        (already present, values identical)
  matched   apricot_preserve      (already present, values identical)
  inserted  32 spreads

Committed. spreads now holds 34 rows.
  nut_butter    14
  seed_butter    6
  fruit_spread  14
```

**Settled.** 34 rows, the two pre-existing spreads recognised rather than duplicated or overwritten, and the category split matches the file.

**The rehearsal has now covered the condition production is actually in**, which is what 002 bought. The script is ready to run against the live database.

## 🧪 Phase 2 — Production Backfill

**2026-04-20.** Fill the production table and prove it matches the file row for row. This is the gate: no read moves to the table until it passes.

### 008 🔧 Backfill production

**Intent.** Run the rehearsed script against production, under live traffic.

**Command.**

```
pnpm backfill-spreads --database pbj_production --dry-run=false
```

**Output.**

```
Reading spread-registry.ts ... 34 spreads
Dual-write flag: ON
Writing to spreads ...

  matched   hazelnut_cocoa        (already present, values identical)
  matched   apricot_preserve      (already present, values identical)
  inserted  32 spreads

Committed in 1.9s. spreads now holds 34 rows.
```

**Settled.** 34 rows in production, and the run behaved identically to the rehearsal — same two matches, same 32 inserts.

**1.9 seconds, row by row, under live traffic.** No order failed during the window and the assembly error rate did not move. That is the payoff of the row-by-row choice in 006 rather than a bulk write.

**What this does not prove.** The script reported success, which means it believes it wrote what the file holds. Whether the table now *contains* that is a separate read, and it is the gate rather than this entry.

---

### 009 🔍 Row-for-row verification — the gate

**Intent.** Compare the two stores independently of the script that wrote them. Read the table directly rather than through `SpreadStore`, because the cache would answer from memory and prove nothing about what was persisted.

**Command.**

```
pnpm verify-spreads --database pbj_production --source file --direct
```

**Output.**

```
Comparing 34 file entries against 34 table rows, column by column.

  33 identical
   1 differs

  ✗ almond_butter
      viscosity   file 0.475   table 0.48
```

**Settled.** **The gate does not pass.** 33 of 34 match; one differs on a single column.

The two counts agreeing is itself worth noting — nothing is missing and nothing is extra, so this is a value problem rather than a completeness problem.

**This is why the gate is a separate read.** Entry 008's script reported a clean run and was not lying: it wrote `0.475`. Something between the write and the read changed the value, which no amount of checking inside the script would have caught.

---

### 010 🔍 One mismatch: viscosity stored at the wrong precision

**Intent.** Find out where `0.475` became `0.48` — in the write, in the column, or in the comparison.

**Command.**

```sql
SELECT column_name, data_type, numeric_precision, numeric_scale
FROM information_schema.columns
WHERE table_name = 'spreads' AND column_name = 'viscosity';

SELECT id, viscosity FROM spreads WHERE id = 'almond_butter';
```

**Output.**

```
 column_name | data_type | numeric_precision | numeric_scale
-------------+-----------+-------------------+---------------
 viscosity   | numeric   |                 3 |             2

 id            | viscosity
---------------+-----------
 almond_butter |      0.48
```

**Settled.** The column is `numeric(3,2)` — two decimal places — and the file carries one spread with three. The database rounded on write, silently and correctly according to its own type.

**The phase 1 migration is where this was decided**, not here. The column was written to mirror `SpreadConfig`, and every spread in the file had two decimals at the time it was read. `almond_butter` was re-measured on 2026-03-11 and has had three ever since.

**Nothing was lost.** The file is still authoritative and still holds `0.475`. What is wrong is the table's ability to hold it.

**Worth carrying.** A type chosen by reading current data encodes the precision the data happened to have that day. This one would not have surfaced until an assembly pressure calculation came out slightly wrong on one spread, months after the cutover, with the file long deleted.

---

### 011 🔧 Correct the column type and re-run the single row

**Intent.** Widen the column, rewrite the one row, and re-run the gate.

**Command.**

```sql
ALTER TABLE spreads ALTER COLUMN viscosity TYPE numeric(4,3);
```

```
pnpm backfill-spreads --database pbj_production --only almond_butter --dry-run=false
pnpm verify-spreads --database pbj_production --source file --direct
```

**Output.**

```
ALTER TABLE

  updated   almond_butter         (viscosity 0.48 -> 0.475)
Committed.

Comparing 34 file entries against 34 table rows, column by column.

  34 identical
   0 differ
```

**Settled.** **The gate passes.** All 34 spreads match the file on every column.

The `ALTER` took 11ms and did not block reads — widening a numeric type rewrites no rows in this engine. Under a different engine this would have been a table rewrite under live traffic and a different plan.

**The `--only` flag was added by this entry**, not by 006. It exists because re-running the whole backfill to fix one row is a larger blast radius than the fix needs.

## 🧪 Phase 3 — Read Cutover

**2026-04-21.** Move reads onto the table one category at a time, watching the assembly error rate between each. Every step is reversible by flipping a flag.

### 012 🔧 Cut `nut_butter` over

**Intent.** Flip the first category. `nut_butter` is chosen because it is the largest at 14 spreads and the most used, so a problem shows up in minutes rather than hours.

**Command.**

```
pnpm flags set registry.read_from_table.nut_butter true --env production
```

**Output.**

```
registry.read_from_table.nut_butter: false -> true
Propagated to 6 of 6 instances in 4s.
```

**Settled.** Nut butter reads now come from the table. The remaining two categories still read the file, which is what makes this a partial cutover rather than a switch.

**The cache is why the effect is not instant.** `SpreadStore` holds a 60-second TTL, so the change is fully in effect a minute after propagation rather than at the moment the flag flips.

---

### 013 🔍 Assembly error rate through the first hour

**Intent.** Establish whether the cutover changed anything the pipeline sees. This is the check the plan's phase 4 gates each category on.

**Command.**

```
pnpm metrics assembly-errors --window 1h --group-by spread_category
```

**Output.**

```
 category      | orders | errors | rate     | baseline (7d)
---------------+--------+--------+----------+---------------
 nut_butter    |   1204 |      2 | 0.166 %  | 0.171 %
 seed_butter   |    311 |      0 | 0.000 %  | 0.012 %
 fruit_spread  |    788 |      1 | 0.127 %  | 0.140 %
```

**Settled.** Nut butter's error rate is 0.166% against a seven-day baseline of 0.171%. Unchanged within noise, over 1,204 orders.

**The other two rows are the control.** They are still reading from the file, and they did not move either — which is what rules out a general problem during the same window and makes the nut butter number attributable.

**What an hour proves and does not.** It proves the read path works and the cache behaves. It does not prove anything about a spread that is only ordered occasionally; three of the 14 nut butters saw no orders in this window and were exercised only by the verification in 011.

---

### 014 🔧 Cut the remaining two categories

**Intent.** Flip `seed_butter` and `fruit_spread`, half an hour apart, with the same check between them.

**Command.**

```
pnpm flags set registry.read_from_table.seed_butter true --env production
# 30 minutes, metrics unchanged
pnpm flags set registry.read_from_table.fruit_spread true --env production
```

**Output.**

```
registry.read_from_table.seed_butter:  false -> true   (6 of 6 instances)
registry.read_from_table.fruit_spread: false -> true   (6 of 6 instances)

 category      | orders | errors | rate     | baseline (7d)
---------------+--------+--------+----------+---------------
 nut_butter    |   2891 |      5 | 0.173 %  | 0.171 %
 seed_butter   |    742 |      1 | 0.135 %  | 0.012 %
 fruit_spread  |   1903 |      2 | 0.105 %  | 0.140 %
```

**Settled.** All three categories read from the table. The operation's end state is met.

**Seed butter's rate looks like a jump and is not.** 0.135% against a 0.012% baseline is one error against 742 orders; the baseline is drawn from a category that averages under 300 orders a day, so a single failure moves it by more than a tenth of a percent. The absolute count — one — is the number that carries meaning here, and the rate is the one that misleads. Checked against the order id: a spread hopper timeout, unrelated to the registry.

**What stays.** The config file is still on disk and still dual-written, and every flag is still flippable. Deleting the file and the dual-write path is phase 5 of the plan, which happens after a full week on the table and is not this operation.

<!-- NEXT ENTRY -->
