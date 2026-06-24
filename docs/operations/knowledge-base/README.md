# Knowledge Base

Reference documentation for how existing systems and processes work. Read these when you need to understand something that's already built — how it's structured, why it works the way it does, and what to know before modifying it.

---

# Two Patterns

Knowledge base docs come in two forms. The distinction matters because they serve different readers at different moments.

**Operational KB (`kb-op`)** — how a system works, step by step, with code references. Written for someone actively working in the codebase who needs to understand a specific mechanism before touching it. Structured as Summary → Flow at a Glance → Parts → Steps, with filenames so the reader can go straight to the code.

**Conceptual KB (`kb-con`)** — what something is and how its pieces relate. Written for someone building a mental model — a new collaborator onboarding, a product decision that requires understanding the architecture, a design review. Structured as Summary → Lifecycle/Model at a Glance → Phases → Failure Cases → Design Decisions. No filenames. The goal is understanding, not navigation.

The test: are you explaining how to navigate through code (operational), or are you explaining how something works as a whole (conceptual)?

---

# Shared Conventions

Both document types share the same formatting conventions:

- `#` for top-level sections, `##` for subsections within them
- `---` between every `#` section
- A **Summary** section as the first `#` heading after the frontmatter
- A **glance block** as the second `#` heading — "Flow at a Glance" for `kb-op`, "Lifecycle/Model at a Glance" for `kb-con`
- Transitionary prose opening every `#` section before the first `##`

The glance format is shared across both KB types and CPF documents intentionally — same visual grammar, different document purpose. A CPF describes something the system *does*. A `kb-con` describes something the system *has*. A `kb-op` describes how to *navigate* something in the codebase.

---

# Examples

| File | Type | What It Covers |
|---|---|---|
| `kb-op_assembly-pipeline.md` | Operational | How the assembly pipeline sequences ingredient prep, bread handling, spread application, and final plating |
| `kb-op_spread-selection-system.md` | Operational | How the spread selector UI works, how availability is enforced, and how selections hand off to assembly |
| `kb-op_order-flow-navigation.md` | Operational | How navigation state works across the order flow wizard, the step lifecycle, and how confirmation validates completeness |
| `kb-op_ingredient-freshness-checks.md` | Operational | How the two-pass freshness check system works — pre-assembly and mid-assembly |
| `kb-con_quality-model.md` | Conceptual | What quality means in the system — the three gates, why they're ordered the way they are, and how to reason about adding new checks |
| `kb-con_order-lifecycle.md` | Conceptual | The states an order moves through from submission to collection, how transitions work, and what the lifecycle is not responsible for |
