# Sandwich Assembly Service

**Type:** Technical

The assembly service is the backend that takes a confirmed sandwich order and produces a finished PB&J. It owns everything from ingredient prep to plating — the machine-side of the product.

## What's Here

```
artifacts/        Implementation notes, decisions, reference material
research/         Problem exploration before a TDD is written
tdd/              Technical design documents, one per feature
procedures/       Repeatable workflows (testing, deployment, incident response)
```

## Current State

Core assembly pipeline is complete and in production — ingredient prep, bread handling, assembly, plating. The quality photo step shipped in the last release.

Active work is **spread pre-handling** — the step that stirs natural spreads and tempers refrigerated ones before assembly. The `SpreadRegistry` model and lookup are done. The `spread-pre-handler.ts` handler is scaffolded but not yet wired into the pipeline.

## To Pick Up the Work

1. Read `tdd/tdd-2026-03-01-spread-pre-handling.md` — implementation plan and acceptance criteria
2. Read `artifacts/assembly-pipeline-overview.md` — where spread pre-handling fits in the full flow
3. Active blocker: handler needs to be wired into `assembly-orchestrator.ts` at Step 9 — exact insertion point is in the TDD

---

## Active Ticket

**ENG-301 — feat: spread pre-handling step**
Steps 1–3 complete (SpreadRegistry model, lookup endpoint, handler scaffold). Step 4 (wire into pipeline) is next. Acceptance criteria in the TDD.
