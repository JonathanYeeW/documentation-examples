# Tickets

A ticket describes a problem before anyone has solved it: what a person can't do today, or has to do the long way, and what should be true once it's fixed. The test is the same whiteboard as a PR description's: someone who knows the product but wasn't in the conversation can read it and explain what's being asked and why.

A ticket says what the product should do, not how to build it. The build is worked out when the ticket is picked up, because the code may have changed by then.

## When to Write One

**Every piece of work that will outlive the conversation it came from gets one.** Its reader is whoever picks it up, and anyone reading it months later.

- **Not a PR description.** A PR description reports what a solution does. A ticket describes the behavior a solution has to reach. See [`../pr-descriptions/`](../pr-descriptions/).
- **Not a plan.** A plan orders the work of building it. A ticket stops at what the product should do.

## Conventions

### Structure

| Section | Holds |
|---|---|
| Title | `[TICKET-ID] type: summary`, conventional commit prefix (`feat:`, `bug:`, `refactor:`, `build:`), all lowercase. The first heading repeats it exactly |
| Title block | `Ticket` |
| `## Context` | What the product is for, where this sits in the journey, then a term bank |
| `## Summary` | The problem as a person meets it, then what's true after the ticket |
| `## <behavior>` | One section per behavior the ticket changes, named for it |
| `## Open questions` | Behavior not yet decided. Left out when there's none |

Nothing else. A section that doesn't fit one of these belongs in the ticket workspace.

### Context

**Context comes before any behavior.** It's what the reader needs in their head to follow the rest.

- **What the product is for** is a bolded label and a short paragraph: the elevator pitch, condensed from the product vision (`pv_` in [`../../strategic/`](../../strategic/)). It anchors the reader in why the product exists, so they can judge the ticket against it. It's the same words in every ticket for that product.
- **Where this sits in the journey** is a bolded label and a few bullets: the screen, what the person is doing there, and how they got there.
- **Terms** is a bolded label and a bullet per term, each a bold word and a one-line definition. It covers any word a reader could misread, including existing product words a reader meeting the feature for the first time wouldn't know. A term the ticket introduces says so.

### Summary

**The Summary opens with the problem, not the product.** One to three sentences on what a person can't do, or what goes wrong, as they'd describe it. Then one sentence starting "After this ticket," on what's true once it's resolved. There's no numbered list: a ticket describes one problem.

### Behaviors

**One section per behavior the ticket changes**, named for it: "Marking a spread out", "What a customer sees". A ticket that changes something for two people, or in two places, has a section for each. A ticket with one behavior has one section.

- **An opener of one or two sentences**, with the person as the subject, saying what they can do or see once the ticket is done.
- **A current and new block**: one fenced block, `Current:` and then `New:`, each a sequence of single actions starting from the same situation. Mark what's new with `← new`. The two should differ only where the ticket does.
- **Bullets for the rules**: what happens at the edges, what stays the same, what collides with something already there. On a ticket these are the behavior it commits to, so a bullet states a decision, not a guess.

A section with no flow of its own, like rules shared by two others, is bullets alone.

### Open questions

**Behavior the ticket can't commit to yet**, each a question a product reader could answer. How to build it is never an open question here; that belongs in the workspace. Something deliberately out of scope is one plain line beneath the questions.

### Writing style

**Tickets are written in product language** ([`writing-styles/product-language/`](../../../writing-styles/product-language/)): the problem as the person meets it, the person as the subject, what they see, and the product's real screen names and labels.

### What stays out

**Anything about how it gets built.** Whoever picks the ticket up works that out from the code as it is then.

- Function, file, service and table names.
- Architecture, data models, and the options considered for them.
- Why the build was chosen. That reasoning goes in the ticket workspace.
- Type, priority and dates. Linear holds them.

## Examples

| File | What It Covers |
|---|---|
| [`tkt_disable-spread-availability.md`](tkt_disable-spread-availability.md) | A feature for two people, staff and customers, with a section for each. The term bank separates the new **Out** from the existing **Out today**, and the Open questions hold what the cold read showed was undecided |
| [`tkt_order-service-registry-fallback.md`](tkt_order-service-registry-fallback.md) | A reliability ticket, where the problem lives in the system but is written as the customer meets it: an error on Start. The staff section has no flow, only a rule, and the Open questions are the product risks of trusting an old copy |
| [`tkt_spread-selector-state-fix.md`](tkt_spread-selector-state-fix.md) | A bug with one behavior and one section. The cause stays out; the Summary names what it costs, a failed order, rather than why it happens |
