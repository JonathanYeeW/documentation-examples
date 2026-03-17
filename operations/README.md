# Operations

Documents that describe how we run the system day-to-day — what exists, how it works, what to do when it breaks, and how to perform repeatable tasks. These are the documents people reach for in the moment: during an incident, while onboarding, or when they need to do something they've only done once before.

## Audience

The team operating the system — engineers on-call, new hires getting up to speed, and anyone who needs to understand or interact with the running product. These assume some technical context but prioritize clarity and speed-of-use over depth.

## Organization

| Subcategory | What It Covers |
|---|---|
| **[knowledge-base/](knowledge-base/)** | Reference docs for how existing systems and processes work |

Postmortems live flat in `operations/` — they are event-based, not reference material.

## Document Types

| Document | Purpose | Subcategory | Status |
|---|---|---|---|
| Knowledge Base | How an existing system or process works — the reference doc you read when you need to understand something that's already built | `knowledge-base/` | ✅ `kb_assembly-pipeline.md`, `kb_spread-selection-system.md` |
| Postmortem | What went wrong, why, and what we're doing about it — the post-mortem that turns a failure into institutional knowledge | flat in `operations/` | ✅ `pm_order-loss-incident.md`, `pm_jelly-overapplication-degradation.md`, `pm_peanut-butter-supplier-outage.md` |
| Runbook | Step-by-step instructions for a repeatable task — the doc you follow when you need to do something correctly and can't afford to improvise | flat in `operations/` | — |
