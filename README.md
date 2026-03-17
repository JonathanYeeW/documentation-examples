<p align="center">
  <img src="assets/brand.png" alt="PB&J Machine Co." width="400" />
</p>

# PB&J Machine Company

A collection of example documents for a fictional software company — all written about the same product, across every layer of company documentation.

## Why This Exists

Most documentation guides tell you *what* to write. They don't show you. This repo is the "show you" part.

Every example is set in the same fictional company: a team building the software interface for a magic peanut butter and jelly sandwich machine. The subject matter is intentionally simple — you already know how a PB&J works. That way the focus stays on the writing, not the domain.

<p align="center">
  <img src="assets/pbj+machine.png" alt="The PB&J Machine" width="600" />
</p>

Each document demonstrates a different document type you'd find in a real software company: product visions, feature specs, technical explorations, and more. Same company, same product, different purposes and audiences.

## Naming Convention

Files follow the pattern `{prefix}_{descriptive-name}.md`.

The prefix is a short identifier for the document type. The suffix describes the specific topic. You can tell what kind of document you're looking at without opening it.

**Recognized document types:**

| Prefix | What It Is |
|---|---|
| `pv` | Product vision — why the company exists and where it's going |
| `fsp` | Feature spec — implementation-ready plan for a specific feature |
| `cpf` | Core product flow — the critical path through a system at any altitude |
| `exp` | Exploration — engineering investigation into a problem space |
| `kb` | Knowledge base — reference doc for how something already works |
| `pm` | Postmortem — what went wrong, why, and what we're doing about it |
| `tkt` | Ticket — scoped unit of work with acceptance criteria and implementation context |
| `sop` | Standard operating procedure — step-by-step procedure for an LLM to follow |
| `rdm-repo` | Repo README — entry point for a codebase; what it does, how to run it, how to contribute |
| `rdm-project` | Project README — LLM orientation doc for an MDP project; what it is, where things live, current state |
| `ctb` | Contributing — architecture overview, dev setup, and testing guidelines for contributors |
| `prd` | PR description — what changed, why, how to review it, and how to test it |
| `rln` | GitHub release notes — what shipped, why it matters, and how to upgrade |

## Organization

Examples are organized into categories that mirror the layers of documentation in a software company — from highest altitude to most operational. READMEs are their own category because they serve two distinct audiences: developers navigating a codebase, and LLMs orienting to a session.

| Category | What It Covers | Examples |
|---|---|---|
| **[strategic/](strategic/)** | Why we exist, where we're going | `pv_pbj-co.md` |
| **[product/](product/)** | What we're building and why | `fsp_artisan-spread-selection.md`, `cpf_sandwich-assembly.md`, `cpf_assembly-pipeline.md` |
| **[engineering/](engineering/)** | How we build it | See `engineering/` — organized into `tickets/`, `shipping/`, `explorations/` |
| **[operations/](operations/)** | How we run it | See `operations/` — `knowledge-base/` for reference docs, flat for postmortems |
| **[communication/](communication/)** | Status, alignment, reflection | — |
| **[llm/](llm/)** | SOPs — procedures an LLM follows as part of a defined workflow | `sop_account-checkin.md` |
| **[readmes/](readmes/)** | README examples organized by audience — repos (for developers) and projects (for LLMs) | See `readmes/` |
| **[developer/](developer/)** | Setup guides and how-to docs for working in a codebase | `kb_vscode-shell-command.md` |

Each category has its own README describing the audience, purpose, and full list of document types — including the ones that haven't been written yet.

## How to Use These

When you're about to write a document, find the matching example here. Read it to see how that document type works in practice — the structure, the altitude, the level of detail, and how it addresses its audience. Then write yours.

## Contributing

This is an evolving collection. New examples get added as new document types are written. If a category shows "—" in the table above, that just means it's next on the list.
