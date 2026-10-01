# Wiki

A wiki article explains a concept the product has: a model, a set of rules, a mechanism. It's written so a reader who knows the product, but not this concept, can understand how it works and the decisions that shape it. It's written in product language, at the altitude of someone who uses the product, not someone reading its code.

It holds what's true now, in present tense, and is updated in place when that changes. A concept that has been settled but not built yet is written the same way. Settling the concept is its own step, and the article is its source of truth.

## When to Write One

**Write one when a concept needs explaining and isn't a flow.** A core product flow (`cpf`) covers a sequence someone walks through. A wiki article covers something the product has, like an order's lifecycle, the quality gates, or how spread availability works.

- **Not a flow.** If the article is a sequence of steps a person takes, it's a `cpf`.
- **Not a feature spec.** A spec (`fsp`) shapes something about to be built, down to its code. A wiki article stays at product altitude and carries no file references.
- **Not a ticket.** A ticket describes a problem and what should be true after it. A wiki article describes how the thing works once it's true.

## Conventions

### Structure

| Section | Holds |
|---|---|
| Title | The concept, in plain words |
| Title block | `Created` |
| `# 📋 Summary` | What the concept is and what it covers, in two or three sentences, including what's covered elsewhere |
| `# 🧭 Context` | What the product is for, where this sits in the journey, the product's terms, and what isn't covered |
| `# ⚙️ Section 1: …` | The model every unit shares, with one worked example |
| `# Section 2: …` onward | The units, grouped by the moment a person meets them |

Summary and Context are unnumbered. Every section after them is a numbered H1 with an emoji ("Section 1: How an Order Moves"), and its H2s are numbered under it ("2.1 Submitted"). The numbers make the two heading levels distinct at a glance.

### Summary

**What the concept is, what it covers, and what's covered elsewhere.** The Summary names the scope up front, so a reader knows before Context whether they're in the right article.

### Context

Context is what a reader needs in their head before the model. It has the same shape as a ticket's or a PR description's Context, plus a scope list.

- **What the product is for** is a bolded label and a short paragraph: the elevator pitch, in the same words the product's tickets use.
- **Where this sits in the journey** is a bolded label and a few bullets: where a person meets the concept, and what they see.
- **Terms** is a bolded label and a bullet per term, in the form `**Word** — definition`. It holds the product's own words only (screen names, labels, actions), because the reader needs them before Section 1.
- **Not covered here** is a bolded label and a bullet per exclusion, with where each one lives.

### The model

**Section 1 is the model every unit shares.** It's where the concept's own words are defined, where they're first used, not in Terms. A reader forgets a glossary by the time the word appears.

- **Every rule lives in the model,** stated as a current rule with its reason in a clause: "ready to collected is the one timed move, because the pickup sensor can't tell a collected sandwich from an abandoned one". There's no separate rules section.
- **One worked example** shows the model working on a real case: one fenced block, with each line labelled the same way.
- **The section ends by naming the moments** the following sections are grouped by, in order.

### Units

**Each section after the model is one moment a person meets the concept**, and opens by saying which moment it is.

- **Every unit has the same shape**, so they can be scanned side by side. What makes a unit distinct comes first, in its opener.
- **A unit's own rules are bullets under its shape.** Rules that hold for every unit are in the model instead.

### Writing style

**Wiki articles are written in product language** ([`writing-styles/product-language/`](../../../writing-styles/product-language/)). Facts go in bullets and reasoning goes in a clause beside the fact.

### What stays out

- **File, function and service names.** They belong in the code, or in a feature spec.
- **How a decision was reached.** What was considered and when it changed goes in the ticket workspace. The article keeps the rule and its reason.
- **Status.** Which parts are built, or when, goes stale. The article describes the concept.
- **A summary table.** Readers skip it, and it repeats the units.

## Examples

| File | What It Covers |
|---|---|
| [`wiki_order-lifecycle.md`](wiki_order-lifecycle.md) | The states an order moves through. The model is how a state changes; the units are the states, grouped by how an order ends |
| [`wiki_quality-model.md`](wiki_quality-model.md) | The three quality gates. The model is failing early to waste less; the units are the gates, in the order a sandwich meets them |
