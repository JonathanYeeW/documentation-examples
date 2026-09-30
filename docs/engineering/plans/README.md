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
3. **📋 Summary** — what the plan covers and how its phases divide, in two or three sentences
4. **🧭 Context** — `Where this sits` and `Terms`, as bullets
5. **🏗️ Implementation** — the phase diagram, the constraints, then a section per phase
6. **📊 Current Status** — a row per phase

Top-level sections are `#` with an emoji, matching the house convention across this collection.

### Decided, verified, or neither

**A plan prescribes only what has been decided or verified.** Anything else is written as an outcome, so the engineer doing the phase chooses the shape after reading the code properly.

- **The test is whether the engineer would be wrong to do it differently.** If deviating breaks a decision, it is a constraint. If deviating is only a different way to build the same thing, it is a guess, and writing it as an instruction takes the choice away from the person best placed to make it.
- **Decisions go in Constraints.** Verified facts go in a `Context` block. Guesses go in neither.
- **A guess worth keeping is labelled as one**, in the `Context` of the phase that will test it. "Starting ideas, to test here rather than take as decided" is the label.
- **A shape that is already settled can be named**, when a TDD, a feature spec or an accepted decision settled it. Say where it was settled.

### Summary

- **What the plan covers, and how its phases divide**: which phases belong to which ticket, or which half builds and which half runs.
- **No list of phase names.** The diagram is the list.
- **Say when later phases are thin on purpose**, because an early phase works out what they depend on.

### Context

**Context is what's true about the problem, as a reader needs it before the phases.** It has two parts, in the same shape as a PR description's Context.

- **`Where this sits`** — bullets on where the work sits in the system or the customer journey, how it behaves today, and why the tickets share one plan. Facts only.
- **The argument for doing the work stays in the document that owns it** — the ticket, the ticket workspace, or an exploration. Context links to it rather than restating it.
- **`Terms`** — every word the constraints and phases use that a reader wouldn't know, including the product's own action names. When two words name one thing, pick one and use it everywhere.
- **A term is defined once.** A phase that needs the code-level version of a term states only the code fact.

### The phase diagram

- **One fenced block, one line per phase**, with a trailing annotation for what is in flight at that phase — the order of a rollout, what reads are still hitting, the row count.
- **Gates are drawn, not described.** A marked line across the block at the point where the work stops if the check fails.
- **Note where rollback stops working.** The phase that is the point of no return is the single most useful thing in the block.
- **It is a diagram, not an index.** Phase names and a few words each.

### Constraints

**A constraint is a rule the whole plan is bound by**, not a fact about one phase. It goes under the diagram, above the phases, because it is what a reader checks a phase against.

- **A constraint is a decision.** A guess about how a phase will be built is not one, however likely it is.
- **State the invariant, not the intention.** "A direct import of the config file anywhere is a bug" is checkable. "We should route through the layer" is not.
- **Use the terms.** A constraint that defines its own words belongs in Terms plus a shorter constraint.
- **Say what stays authoritative during the change**, when two stores exist at once. Ambiguity there is what makes a mismatch unresolvable.
- **A performance property belongs here if it constrains a phase's design.** Caching on the hot path is a constraint on phase 2, not an optimization someone gets to defer.

### Phases

- **The opener is what the phase does, then why it sits where it does**, in two sentences at most. It is the part of the phase a reader actually reads, so it carries the ordering argument, and there is no separate section for it.
- **Checklist items are outcomes.** "A leftover can be told apart from a fresh plan", not "add a marker field to `Dish`". An outcome is still something a second person can confirm. `[x]` for done.
- **Phases further out stay thin.** A phase gets detail when the phase before it finishes, because that is when what it depends on is known. The plan is edited as it goes, and that is when to add it.
- **A decision made during a phase is recorded in the document that owns it**, and the checklist item says so: "decided and recorded in the recommendations document".
- **The last item of a phase is usually its check** — the confirmation, the review, the comparison run. That is the item the next phase gates on.
- **A phase ends with a `Context` block when it has lookup values.** Only verified ones: table and column names, flags and their values, file paths, expected counts, build order. These are things a reader copies rather than reads. A phase with nothing to look up has no block.
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
| `plan_spread-nutritional-backfill.md` | Build-then-run: four phases where each is scaffolding for the next, gated on a hand review of the first phase's output. It carries a live blocker in a phase `Context`, an unsettled data format that would force a re-run, and a starting idea labelled for the phase that tests it |
| `plan_spread-registry-database-migration.md` | Migration under live traffic: five phases, a verification gate, a flag-based rollback across phase 4, and a phase 5 marked as the point of no return. It shows constraints doing real work (what stays authoritative, and what counts as a bug), and names a settled shape along with where it was settled |
