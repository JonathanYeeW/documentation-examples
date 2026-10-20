# Core Product Flows

A core product flow is the critical path through a system, written as an ordered walk from a defined start state to a defined end state. It exists to anchor — a reader who knows the rules but cannot picture how the pieces connect reads this and can then place every rule they already knew.

Altitude is not prescribed. It is set entirely by where the start and end points sit. A flow that starts with a user approaching a machine and ends with a sandwich in their hand is a product journey; one that starts with a function call and ends with a return value is a pipeline. Same structure, different zoom.

## When to Use One

**Write one when the rules exist and the shape does not.** A standard says what is true at each hop; a flow says what the hops are and what happens between them. A reader with only the standard knows every rule and still cannot say where a request died.

- **The test is whether someone can name the boundaries.** If a knowledgeable reader cannot list the steps in order, a flow is missing.
- **Pick the start and end points first.** They are the only altitude decision, and everything else follows from them. A flow whose start point drifts partway through is two flows.
- **One flow, not a survey.** The critical path only. Alternate paths, error branches, and edge cases are noted where they leave the path, not followed.
- **A boundary is a good place to stop.** When the path crosses into a system another document owns, hand off rather than continuing — two flows meeting at a boundary beat one that spans it and belongs to neither.
- **Not a feature spec.** A spec describes something being built. A flow describes something that already works.

## Conventions

### Structure

1. **Title** — `Core Product Flow — <start> to <end>`
2. **Title block** — `Created`, `Updated`, and a `Context` line stating the start and end points
3. **Summary** — the shape of the flow in one sentence, what a reader is likely to get wrong about it, and when they need it
4. **Flow at a Glance** — the whole path in one fenced block
5. **Phases** — one section each, with numbered steps
6. **A closing section** — error handling, or a symptom table, or whatever the flow's failures need

### What travels

A flow is easiest to follow when the reader can name the thing being passed along. An order, a request, a token: whatever crosses from one actor to the next.

- **Name it once and keep the name.** The step that creates it says what it is called, and every later step uses that word.
- **Show what it says, once, where it is created.** Its contents as the claims it makes, with real-looking values — "white bread, toasted; almond butter; grape jelly" — not a schema.
- **Say what it leaves out** when a reader would assume it carries more, such as "nothing about who placed it".

### The glance block

- **Every step appears, numbered continuously across phases.** A reader should be able to hold the whole path from this block alone.
- **Phases are separated by an explicit handoff line**, naming what is taking over and what crosses — `handoff to the machine: the order`.
- **Annotate the block with what a failure does** at the points where a failure leaves the path.
- **It is a diagram, not an index.** Steps carry a few words, not sentences.

### Steps

- **A bolded step title, one or two sentences, then bullets.** Numbered to match the glance block exactly. The sentences say what the step is and what it decides; the bullets carry the facts. A reader skims the bullets and reads the opener only when they need to.
- **A step with a single fact is a sentence.** Do not bullet one item. Two examples per collection is about right — they teach that the shape is a tool rather than a template.
- **A different kind of item inside the list gets a bolded label.** A scar (`**INC-042**`), a caveat, an optimization note. The label says this line is not another fact of the same sort, and it costs nothing — no sub-heading, no separate block.
- **No sub-headings inside a step.** If something seems to need one, it is a step.
- **A lone argument stays prose, under the list.** Reasoning bulleted on its own reads as a task. Facts are bullets; the argument about them is a sentence underneath.
- **The step title names who acts.** "The machine readies the spread", not "Spread pre-handling". When actors trade off step by step, the title is where the reader sees the trade.
- **Say what is decided at each step, not just what occurs.** A step that only narrates is a step a reader already inferred from the block.
- **Write a check as the question it answers.** "Has the spread mixed?" says what is being decided; "mix validation" does not. When another document owns the check, use its name and its question, and point to it.
- **Say what gets through without the check**, whether or not an incident is behind it. A check whose purpose isn't written down is easy to remove by mistake.
- **State what the flow never does** wherever a reader would assume it does. "No login and no account" corrects a model the reader brought with them.
- **Describe a term before naming it.** "The Spread Registry, the list of spreads the machine knows" gives the reader the thing and its name at once. No glossary.
- **Name the failure the step is guarding against**, and what it costs when it fires. A quality gate's position in the sequence is a claim about what is cheap to waste.
- **Reference a real incident where one shaped the design.** A rule with a scar attached does not get optimized away by the next person. Reference it — `INC-042` — rather than narrating when it happened and what was learned.
- **Say where a fact lives rather than restating it.** A value repeated from the standard that owns it is a value with two places to go stale.

### Headings

Top-level sections are `##` with no emoji, unlike the rest of this collection. A flow is read while looking at code or at a broken email, and the headings are navigation rather than decoration.

### Closing

**Every flow ends with what happens when it does not complete.** How that is written depends on the flow:

- **An error-handling section** when failures fall into classes with different costs.
- **A symptom table** when the reader's real question is diagnostic — a row per symptom, the step it died at, and what that means. This is the strongest ending for a flow whose failures are observed from outside.
- **A failure table** when the checks are the point — a row per check, keyed by the question it failed and what has been used up by then. The cost column is what makes the order of the checks readable.

## Examples

| File | What It Covers |
|---|---|
| `cpf_sandwich-assembly.md` | Product altitude — a customer approaching a machine to a sandwich in their hand, three phases across two actors. Shows the order as the thing that travels, step titles that name who acts, checks written as questions, and a failure table ending. Step 4 is a bare sentence because it carries one fact |
| `cpf_assembly-pipeline.md` | Pipeline altitude — one function call to its return, conditional branches driven by configuration, ending in an error-handling table that classifies failures by what they waste |
