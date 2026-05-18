# Knowledge Base

Reference documentation for how existing systems and processes work. Read these when you need to understand something that's already built — how it's structured, why it works the way it does, and what to know before modifying it.

## Two Patterns

Knowledge base docs come in two forms. The distinction matters because they serve different readers at different moments.

**Operational KB** — how a system works, step by step, with code references. Written for someone actively working in the codebase who needs to understand a specific mechanism before touching it. Structured as parts → steps → implementation details, with filenames so the reader can go straight to the code.

**Conceptual KB** — what something is and how its pieces relate. Written for someone building a mental model — a new collaborator onboarding, a product decision that requires understanding the architecture, a design review. No steps, no filenames. The goal is understanding, not navigation.

The test: are you explaining how to navigate through something (operational), or are you explaining how something works as a whole (conceptual)?

## Examples

| File | Type | What It Covers |
|---|---|---|
| `kb-op_assembly-pipeline.md` | Operational | How the assembly pipeline sequences ingredient prep, bread handling, spread application, and final plating |
| `kb-op_spread-selection-system.md` | Operational | How the spread selector UI works, how availability is enforced, and how selections hand off to assembly |
| `kb-op_order-flow-navigation.md` | Operational | How navigation state works across the order flow wizard, the step lifecycle, and how confirmation validates completeness |
| `kb-op_ingredient-freshness-checks.md` | Operational | How the two-pass freshness check system works — pre-assembly and mid-assembly |
| `kb-con_quality-model.md` | Conceptual | What quality means in the system — the three gates, why they're ordered the way they are, and how to reason about adding new checks |
| `kb-con_order-lifecycle.md` | Conceptual | The states an order moves through from submission to collection, how transitions work, and what the lifecycle is not responsible for |
