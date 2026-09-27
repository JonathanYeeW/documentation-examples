# Engineering

Documents that describe how we build things — architecture, implementation decisions, and the technical context an engineer needs to contribute effectively. These are the deepest-altitude documents in the company. They assume technical fluency and prioritize precision over accessibility.

## Audience

Engineers. A new engineer joining the team should be able to read the engineering docs and understand how the system works, why it was built that way, and where to make changes without breaking things.

## Organization

Engineering examples are organized into six subcategories:

| Subcategory | What It Covers |
|---|---|
| **[tickets/](tickets/)** | Scoped units of work — what needs to be built or fixed, and why |
| **[plans/](plans/)** | The sequence and scope of a multi-phase body of work, and where it currently stands |
| **[pr-descriptions/](pr-descriptions/)** | What a change set does for the user and why, written for anyone who knows the product |
| **[shipping/](shipping/)** | Release notes — what shipped across a release, for people outside the team |
| **[explorations/](explorations/)** | Technical investigations before committing to a solution |
| **[notebooks/](notebooks/)** | Records of hands-on operations — what was run, what came back, and what it settled |

READMEs and contributing guides that live inside codebases are in [`readmes/repos/`](../readmes/repos/).

## Document Types

| Document | Purpose | Subcategory | Status |
|---|---|---|---|
| Ticket | A scoped unit of work — what's broken or what needs to be built, why it matters, acceptance criteria, and enough implementation context to pick it up without a meeting. Titles follow conventional commit prefix pattern (`feat:`, `bug:`, `refactor:`, `build:`, etc.), all lowercase. | `tickets/` | ✅ `tkt_spread-selector-state-fix.md`, `tkt_disable-spread-availability.md`, `tkt_order-service-registry-fallback.md` |
| PR Description | What changed for the user and why, anchored in the customer journey, so anyone who knows the product can explain and defend the change without reading the code. | `pr-descriptions/` | ✅ `prd_artisan-spread-selection.md` |
| GitHub Release Notes | What shipped in a release, why it matters, and how to upgrade — written for developers watching the repo. Published alongside a tagged release. | `shipping/` | ✅ `rln_artisan-spread-selection.md` |
| Exploration | Engineering investigation into a problem space — what's happening, why, and what the options are, before committing to a solution. | `explorations/` | ✅ `exp_spread-selection-reset.md` |
| Technical Design Doc (TDD) | Detailed technical plan for a complex system change — problem framing, approach, alternatives considered, and implementation design. | `explorations/` | — |
| Notebook | The record of one hands-on operation — provisioning, a migration, a live configuration change — written entry by entry as it happens. Typed entries (action, observation, decision, file), verbatim output, failures retained, and a resource register of what exists now. Covers one operation with a clear start and end, not one ticket and not one phase. | `notebooks/` | ✅ `nb_order-service-provisioning.md`, `nb_spread-registry-backfill.md` |
| Implementation Plan | The sequence and scope of a body of work — what's being built, in what order, what's in and out, and where it currently stands. Answers *what happens and in what order*, not *how each piece is shaped*. | `plans/` | ✅ `plan_spread-nutritional-backfill.md`, `plan_spread-registry-database-migration.md` |
