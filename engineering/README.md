# Engineering

Documents that describe how we build things — architecture, implementation decisions, and the technical context an engineer needs to contribute effectively. These are the deepest-altitude documents in the company. They assume technical fluency and prioritize precision over accessibility.

## Audience

Engineers. These are written by engineers for engineers. A new engineer joining the team should be able to read the engineering docs and understand how the system works, why it was built that way, and where to make changes without breaking things.

## Document Types

| Document | Purpose | Status |
|---|---|---|
| Exploration | Engineering investigation into a problem space — what's happening, why, and what the options are, before committing to a solution | ✅ `exp_spread-selection-reset.md` |
| Ticket | A scoped unit of work — what's broken or what needs to be built, why it matters, acceptance criteria, and enough implementation context to pick it up without a meeting. Titles follow the conventional commit prefix pattern (`feat:`, `bug:`, `refactor:`, `build:`, etc.) and are all lowercase. | ✅ `tkt_spread-selector-state-fix.md`, `tkt_disable-spread-availability.md`, `tkt_order-service-registry-fallback.md` |
| Repo README | The entry point for a codebase — what it does, how to run it, and how to contribute. One per repo. Comes in five flavors depending on repo type: CLI, API, MCP server, web app, mobile app. | ✅ `rdm_cli-pbj.md`, `rdm_api-pbj.md`, `rdm_mcp-pbj.md`, `rdm_web-pbj.md`, `rdm_mobile-pbj.md` |
| Contributing | Architecture overview, directory structure, local dev setup, and testing guidelines for contributors. Lives at `CONTRIBUTING.md` in the repo root. Referenced from the README. | ✅ `ctb_cli-pbj.md` |
| Technical Design Doc (TDD) | Detailed technical plan for a complex system change — problem framing, approach, alternatives considered, and implementation design | — |
| Architecture Overview | How the system is structured at a high level — services, data flow, dependencies, and the key decisions that shaped the architecture | — |
| PR Description | What changed, why, how to review it, and how to test it — the context a reviewer needs to evaluate a code change | — |
