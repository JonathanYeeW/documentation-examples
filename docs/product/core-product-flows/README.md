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
3. **Summary** — what the flow covers, its phases, and when a reader needs it
4. **Flow at a Glance** — the whole path in one fenced block
5. **Phases** — one section each, with numbered steps
6. **A closing section** — error handling, or a symptom table, or whatever the flow's failures need

### The glance block

- **Every step appears, numbered continuously across phases.** A reader should be able to hold the whole path from this block alone.
- **Phases are separated by an explicit handoff line**, naming what is taking over.
- **Annotate the block with what a failure does** at the points where a failure leaves the path.
- **It is a diagram, not an index.** Steps carry a few words, not sentences.

### Steps

- **One bolded step title, then what happens.** Numbered to match the glance block exactly.
- **Say what is decided at each step, not just what occurs.** A step that only narrates is a step a reader already inferred from the block.
- **Name the failure the step is guarding against**, and what it costs when it fires. A quality gate's position in the sequence is a claim about what is cheap to waste.
- **Reference a real incident where one shaped the design.** A rule with a scar attached does not get optimized away by the next person.
- **Say where a fact lives rather than restating it.** A value repeated from the standard that owns it is a value with two places to go stale.

### Closing

**Every flow ends with what happens when it does not complete.** How that is written depends on the flow:

- **An error-handling section** when failures fall into classes with different costs.
- **A symptom table** when the reader's real question is diagnostic — a row per symptom, the step it died at, and what that means. This is the strongest ending for a flow whose failures are observed from outside.

## Examples

| File | What It Covers |
|---|---|
| `cpf_sandwich-assembly.md` | Product altitude — a user approaching a machine to a sandwich in their hand, three phases across two actors, ending in metrics |
| `cpf_assembly-pipeline.md` | Pipeline altitude — one function call to its return, conditional branches driven by configuration, ending in an error-handling section that classifies failures by what they waste |
