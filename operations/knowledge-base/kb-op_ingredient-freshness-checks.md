# Ingredient Freshness Checks

**Created:** 2026-04-11
**Last Updated:** 2026-04-11
**Directory:** `sandwich-service/quality`

Freshness checks run at two points in the assembly pipeline: when ingredients are pulled from storage (pre-assembly), and again immediately before spread application (mid-assembly). The two-pass design is intentional — the pre-assembly check catches obvious failures before any assembly work begins, and the mid-assembly check catches edge cases that only become visible after prep (e.g., bread that looked fine but dried out during toasting). Understanding both passes is important before modifying the quality check system or changing how failures are surfaced to users.

Read this when you're working on ingredient handling, quality reporting, or any part of the pipeline that interacts with the freshness system.

## Part 1: 🔍 Pre-Assembly Check

Runs immediately after ingredients are pulled from storage and staged at the prep station. If anything fails here, the order fails before a single step of assembly runs.

**Step 1: Run checks on all ingredients in parallel**
`freshness-service.ts`

`checkFreshness()` accepts an array of staged ingredients and runs all checks in parallel. Checks are independent — a stale jelly finding doesn't prevent the bread check from completing. The results are collected and evaluated together after all checks finish. Running in parallel keeps the pre-assembly check fast; running checks serially would add 1–2 seconds per ingredient for orders with multiple items.

**Step 2: Evaluate bread freshness**
`freshness-service.ts → checkBread()`

Bread freshness is determined by two signals: the `bestByDate` on the packaging and the moisture sensor reading at the prep station. A loaf past its best-by date fails immediately. A loaf within its date but with a moisture reading below threshold (indicating it's dried out) also fails. Both signals must pass for bread to clear. The moisture threshold is defined in `freshness-config.ts` and is calibrated per bread type — sourdough has a lower threshold than white bread because it's denser.

**Step 3: Evaluate spread freshness**
`freshness-service.ts → checkSpread()`

Spread freshness checks the `openedDate` stored in the inventory system when the container was first used. Once opened, peanut butter is considered fresh for 90 days; almond butter for 60 days; sunflower butter for 45 days. If the container has no `openedDate` (never opened), it passes automatically and the `openedDate` is stamped at this step. Per-spread freshness windows are defined in the spread registry in `spread-registry.ts`.

**Step 4: Evaluate jelly freshness**
`freshness-service.ts → checkJelly()`

Same `openedDate` pattern as spread. All jelly varieties share a 30-day freshness window after opening. Jelly also runs a visual check via the jar sensor — if the sensor detects surface mold indicators, the jar fails regardless of date. The visual check is jelly-specific; it doesn't run on spreads.

**Step 5: Fail fast on any failure**
`freshness-service.ts`

If any ingredient fails, `checkFreshness()` returns `{ passed: false, failures: IngredientFailure[] }`. The pipeline fails the order immediately and surfaces a user-facing message naming the failed ingredient. The user is prompted to try again — if the issue is systemic (e.g., all jelly jars in storage are expired), every retry will fail until operations resolves the inventory issue.

## Part 2: 🧪 Mid-Assembly Check

A second, lighter check runs after bread prep and before spread application. It catches degradation that happened during prep — primarily toasting-related drying.

**Step 1: Check bread post-toasting**
`bread-handler.ts → postToastFreshnessCheck()`

After the 30-second cool-down period for toasted orders, the moisture sensor runs a second reading. Some bread types lose enough moisture during toasting to fall below the freshness threshold even if they passed the pre-assembly check. If this check fails, the order fails with a specific `TOAST_DRIED_BREAD` failure code — distinct from the generic bread freshness failure so operations can track toasting-related failures separately in the quality dashboard.

**Step 2: Skip for untoasted orders**
`bread-handler.ts`

The mid-assembly check only runs for toasted orders. Untoasted bread that passed the pre-assembly check is assumed to be in the same condition — no second read needed. If you add a new bread prep step that could affect moisture levels, add a corresponding mid-assembly check in that handler.

---

*Freshness checks run in two passes — a parallel pre-assembly check that catches failures before any assembly work begins, and a targeted mid-assembly check that catches toasting-related degradation after bread prep. Any failure at either pass stops the order immediately.*
