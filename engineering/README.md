# Engineering

Documents that describe how we build things — architecture, implementation decisions, and the technical context an engineer needs to contribute effectively. These are the deepest-altitude documents in the company. They assume technical fluency and prioritize precision over accessibility.

## Audience

Engineers. These are written by engineers for engineers. A new engineer joining the team should be able to read the engineering docs and understand how the system works, why it was built that way, and where to make changes without breaking things.

## Organization

Engineering examples are organized into four subcategories:

| Subcategory | What It Covers |
|---|---|
| **[repos/](repos/)** | Documents that live inside a codebase — READMEs and contributing guides |
| **[tickets/](tickets/)** | Scoped units of work — what needs to be built or fixed, and why |
| **[shipping/](shipping/)** | Output artifacts of shipping a feature — PR descriptions and release notes |
| **[explorations/](explorations/)** | Technical investigations before committing to a solution |

## Document Types

| Document | Purpose | Subcategory | Status |
|---|---|---|---|
| Repo README | The entry point for a codebase — what it does, how to run it, and how to contribute. One per repo. Comes in five flavors: CLI, API, MCP server, web app, mobile app. | `repos/` | ✅ `rdm_cli-pbj.md`, `rdm_api-pbj.md`, `rdm_mcp-pbj.md`, `rdm_web-pbj.md`, `rdm_mobile-pbj.md` |
| Contributing | Architecture overview, directory structure, local dev setup, and testing guidelines for contributors. Lives at `CONTRIBUTING.md` in the repo root. | `repos/` | ✅ `ctb_cli-pbj.md` |
| Ticket | A scoped unit of work — what's broken or what needs to be built, why it matters, acceptance criteria, and enough implementation context to pick it up without a meeting. Titles follow conventional commit prefix pattern (`feat:`, `bug:`, `refactor:`, `build:`, etc.), all lowercase. | `tickets/` | ✅ `tkt_spread-selector-state-fix.md`, `tkt_disable-spread-availability.md`, `tkt_order-service-registry-fallback.md` |
| PR Description | What changed, why, how to review it, and how to test it — the context a reviewer needs to evaluate a code change without a meeting. Written by the author, read by the reviewer before they look at the diff. | `shipping/` | ✅ `prd_artisan-spread-selection.md` |
| GitHub Release Notes | What shipped in a release, why it matters, and how to upgrade — written for developers watching the repo. Published alongside a tagged release. | `shipping/` | ✅ `rln_artisan-spread-selection.md` |
| Exploration | Engineering investigation into a problem space — what's happening, why, and what the options are, before committing to a solution. | `explorations/` | ✅ `exp_spread-selection-reset.md` |
| Technical Design Doc (TDD) | Detailed technical plan for a complex system change — problem framing, approach, alternatives considered, and implementation design. | `explorations/` | — |
| Architecture Overview | How the system is structured at a high level — services, data flow, dependencies, and the key decisions that shaped the architecture. | `explorations/` | — |
