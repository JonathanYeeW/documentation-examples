# Plans

An implementation plan is the sequence and scope of a body of work — what is being built, in what order, what is in and out, and where it currently stands. It answers *what happens and in what order*, not *how each piece is shaped*.

It is the working document of a multi-session effort. It is written before the work starts, read at the top of every session that continues it, and edited as phases complete. A plan that is only ever written and never updated has been used as a proposal, which is a different document.

## When to Use One

**Write one when the work has an order that matters.** If the steps could be done in any sequence, a ticket holds them. A plan exists because phase 3 is unsafe before phase 2, and someone has to be able to see that without reconstructing it.

- **The test is whether a step is gated.** If completing one step is what makes the next one safe, that dependency is the plan's reason to exist.
- **Phases are units of state, not units of effort.** A phase ends where the system is in a coherent condition someone could stop at. Splitting by how much work fits in a day produces phases nobody can gate on.
- **In and out of scope is part of the plan, not a preamble.** The constraint that says which store is authoritative is what makes a reviewer able to catch a wrong step.
- **One plan per body of work, across however many tickets it takes.** The ticket is a unit of assignment; the plan is the arc.
- **Not an exploration.** An exploration decides what to do. A plan assumes that is settled and sequences it. A plan whose approach is still being argued is an exploration wearing phase headings.
- **Not a TDD.** A TDD shapes one component — signatures, data model, alternatives. A plan orders the components and links to the TDDs that shape them.

## Conventions

### Structure

1. **Title** — the body of work, in plain words
2. **Title block** — `Created`, `Ticket`, `Codebase`
3. **📋 Summary** — what the plan covers and the phases by name, in one paragraph
4. **🧭 Approach** — why the work is sequenced this way, as numbered prose
5. **🏗️ Implementation** — the phase diagram, the constraints, then a section per phase
6. **📊 Current Status** — a row per phase

Top-level sections are `#` with an emoji, matching the house convention across this collection.

### Approach

- **Numbered prose, one entry per phase.** A bolded sentence saying what the phase does, then why it goes there. The why is the whole section — a reader who only wanted the order already had it from the diagram.
- **Name what makes the next phase safe.** "Dual-write, so the table and the file never disagree" states the step and the property it establishes in one line.
- **Say what the work is replacing and why the replacement is needed**, once, at the top. That is the only place the motivation appears.

### The phase diagram

- **One fenced block, one line per phase**, with a trailing annotation for what is in flight at that phase — the order of a rollout, what reads are still hitting, the row count.
- **Gates are drawn, not described.** A marked line across the block at the point where the work stops if the check fails.
- **Note where rollback stops working.** The phase that is the point of no return is the single most useful thing in the block.
- **It is a diagram, not an index.** Phase names and a few words each.

### Constraints

**A constraint is a rule the whole plan is bound by**, not a fact about one phase. It goes under the diagram, above the phases, because it is what a reader checks a phase against.

- **State the invariant, not the intention.** "A direct import of the config file anywhere is a bug" is checkable. "We should route through the layer" is not.
- **Say what stays authoritative during the change**, when two stores exist at once. Ambiguity there is what makes a mismatch unresolvable.
- **A performance property belongs here if it constrains a phase's design.** Caching on the hot path is a constraint on phase 2, not an optimization someone gets to defer.

### Phases

- **A one-sentence opener stating what the phase does**, then a checklist.
- **Checklist items are verifiable.** Each is a thing that is either done or not, written so a second person could confirm it. `[x]` for done.
- **The last item of a phase is usually its check** — the confirmation, the review, the comparison run. That is the item the next phase gates on.
- **Every phase ends with a `Context` block.** Lookup values only: table and column names, flags and their values, file paths, expected counts, build order. Things a reader copies rather than reads.
- **A blocker lives in the `Context` of the phase it blocks**, stated with the options for clearing it. That is where the person doing the work will be looking, and it keeps the plan from needing a separate risks section.
- **Expected counts are written down before the phase runs.** "34 rows — 18, 9, 7" is what makes a result checkable instead of plausible.

### Current Status

**The status table is the reason the file is reopened.** A row per phase: state, and a note saying what it is waiting on or what the last run reported.

- **The note carries the specifics.** "Verification reports 2 mismatches on `viscosity`" is the whole point of the row; "in progress" alone is not.
- **A not-started row says what it is gated on**, not just that it has not begun.
- **It is updated as phases move, not rewritten at the end.** A plan whose status table is stale has stopped being the working document and become a record of intent.

## Examples

| File | What It Covers |
|---|---|
| `plan_spread-nutritional-backfill.md` | Build-then-run — four phases where each is scaffolding for the next, gated on a hand review of the first phase's output. Carries a live blocker in a phase `Context` and an unsettled data format that would force a re-run |
| `plan_spread-registry-database-migration.md` | Migration under live traffic — five phases, a verification gate, a flag-based rollback across phase 4, and a phase 5 marked as the point of no return. Shows constraints doing real work: what stays authoritative, and what counts as a bug |
