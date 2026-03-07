# documentation-examples

A collection of example documents for a fictional software company — all written about the same product, across every layer of company documentation.

## Why This Exists

Most documentation guides tell you *what* to write. They don't show you. This repo is the "show you" part.

Every example is set in the same fictional company: a team building the software interface for a magic peanut butter and jelly sandwich machine. The subject matter is intentionally simple — you already know how a PB&J works. That way the focus stays on the writing, not the domain.

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
| `prep` | Prep doc — LLM-synthesized briefing produced immediately before a recurring interaction |

## Organization

Examples are organized into five categories that mirror the layers of documentation in a software company — from highest altitude to most operational.

| Category | What It Covers | Examples |
|---|---|---|
| **[strategic/](strategic/)** | Why we exist, where we're going | `pv_pbj-co.md` |
| **[product/](product/)** | What we're building and why | `fsp_artisan-spread-selection.md`, `cpf_sandwich-assembly.md`, `cpf_assembly-pipeline.md` |
| **[engineering/](engineering/)** | How we build it | `exp_spread-selection-reset.md`, `tkt_spread-selector-state-fix.md` |
| **[operations/](operations/)** | How we run it | `kb_assembly-pipeline.md` |
| **[communication/](communication/)** | Status, alignment, reflection | — |
| **[llm/](llm/)** | Documents produced by an LLM as part of a defined workflow | `prep_account-checkin.md` |

Each category has its own README describing the audience, purpose, and full list of document types — including the ones that haven't been written yet.

## How to Use These

When you're about to write a document, find the matching example here. Read it to see how that document type works in practice — the structure, the altitude, the level of detail, and how it addresses its audience. Then write yours.

## Contributing

This is an evolving collection. New examples get added as new document types are written. If a category shows "—" in the table above, that just means it's next on the list.
