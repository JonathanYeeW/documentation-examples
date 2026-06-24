# Engineering

Documents that describe how we build things — architecture, implementation decisions, and the technical context an engineer needs to contribute effectively. These are the deepest-altitude documents in the company. They assume technical fluency and prioritize precision over accessibility.

## Audience

Engineers. A new engineer joining the team should be able to read the engineering docs and understand how the system works, why it was built that way, and where to make changes without breaking things.

## Organization

Engineering examples are organized into three subcategories:

| Subcategory | What It Covers |
|---|---|
| **[tickets/](tickets/)** | Scoped units of work — what needs to be built or fixed, and why |
| **[shipping/](shipping/)** | Output artifacts of shipping a feature — PR descriptions and release notes |
| **[explorations/](explorations/)** | Technical investigations before committing to a solution |

READMEs and contributing guides that live inside codebases are in [`readmes/repos/`](../readmes/repos/).

## Document Types

| Document | Purpose | Subcategory | Status |
|---|---|---|---|
| Ticket | A scoped unit of work — what's broken or what needs to be built, why it matters, acceptance criteria, and enough implementation context to pick it up without a meeting. Titles follow conventional commit prefix pattern (`feat:`, `bug:`, `refactor:`, `build:`, etc.), all lowercase. | `tickets/` | ✅ `tkt_spread-selector-state-fix.md`, `tkt_disable-spread-availability.md`, `tkt_order-service-registry-fallback.md` |
| PR Description | What changed, why, how to review it, and how to test it — the context a reviewer needs to evaluate a code change without a meeting. Written by the author, read by the reviewer before they look at the diff. | `shipping/` | ✅ `prd_artisan-spread-selection.md` |
| GitHub Release Notes | What shipped in a release, why it matters, and how to upgrade — written for developers watching the repo. Published alongside a tagged release. | `shipping/` | ✅ `rln_artisan-spread-selection.md` |
| Exploration | Engineering investigation into a problem space — what's happening, why, and what the options are, before committing to a solution. | `explorations/` | ✅ `exp_spread-selection-reset.md` |
| Technical Design Doc (TDD) | Detailed technical plan for a complex system change — problem framing, approach, alternatives considered, and implementation design. | `explorations/` | — |
| Implementation Plan | Agreed execution plan captured mid-work, after key architectural decisions are locked but before all phases are complete. Covers what's being built, the phased approach, current status, and open questions. | `engineering/` (flat) | ✔️ `plan_spread-nutritional-backfill.md` |
