# SOP: Document Review & Rewrite

**Purpose:** Rewrite a document against a fixed standard. Applies to any document meant to be read across sessions by both people and models. There is always an input document — this is not a writing-from-scratch procedure.

---

## Trigger

A document is provided with the instruction to review it against this SOP.

Begin Phase 1.

---

## The Principle

A document holds what is true now. It does not hold the reasoning that produced it.

Facts go in the document. The reader does the interpretation. Why something was chosen, what was tried first, what changed and when — that lives in the ticket workspace or the session log, not here.

Everything below follows from this and from the document's stated purpose.

---

## Phase 1 — Establish Context

Do not review for style yet. Ask:

1. **What is this document for?** What it is supposed to do for someone who opens it cold.
2. **Who reads it, and what do they need from it?**
3. **What do you expect to find in it?** The sections they would expect, in the order they would expect them.

While waiting, do two things.

**Read the document in full** and identify the standing claims — the facts it exists to convey.

**Find the reference example.** Name the document type, then read the matching example in `plugins/documentation-examples/`. The example is what the rewrite is shaped against — its sections, its altitude, how it handles repeated units. A draft written without reading it will be coherent and still wrong on form, and the correction costs a full rewrite. If no example exists for this type, say so: the run is then establishing the shape rather than matching one, and Phase 4 has more to do.

Restate the purpose in one sentence and get agreement. That sentence is the test every section is measured against for the rest of the run.

> ⏸️ **GATE → User.** Purpose and reference example are both confirmed before any rewriting happens. A run without either makes bad cuts.

---

## Phase 2 — Draft the Rewrite

Apply the standard and produce the rewritten document. Do not produce a findings list, a critique, or a list of proposed changes — the opinions go into the draft. The draft is the argument.

Save it as a new file alongside the original. That file is the workspace for the rest of the run — every later change is applied to it in place. The original is never modified.

Rules for the draft:

- **Preserve every standing claim** identified in Phase 1. Compression is not deletion of content — a fact in the input is in the output unless the standard cut it deliberately.
- **Do not add.** No new framing, examples, or sections. Missing content is raised as a question, not filled in.
- **Structure may change** when the structure is the problem — moving reference content out of a sequence, merging sections that repeat each other, splitting one that buries a claim.
- **Cut content moves, it does not vanish.** When reasoning worth keeping is removed, name where it should go (ticket workspace, session log) so it can be moved.

Deliver the draft with a short note — three lines or fewer:

- Length before → after
- What drove most of the change
- Anything cut that needs a home elsewhere

> ⏸️ **GATE → User.** The draft is read against the original.

---

## Phase 3 — Review and Iterate

The user reacts to the draft. Apply changes directly to the workspace file and re-deliver.

Expect several full rewrites before form settles. The first draft usually gets content fit roughly right and form wrong, so a rewrite driven entirely by form is the normal path rather than a failure.

Reactions to form and presentation are where the standard is least settled. When a reaction contradicts the standard, the standard is what changes — capture it as a new or amended criterion rather than treating it as a one-off correction.

Loop until the user accepts. On acceptance, the draft replaces the original.

> ⏸️ **GATE → User.** The user accepts the draft before anything is folded back.

---

## Phase 4 — Fold Corrections Back

A correction that only fixes one document is a correction that gets made again. Every accepted change is a candidate for the standard.

Go through the changes the user asked for in Phase 3 and sort them:

- **General** — the change would apply to any document of this type. Update the matching example in `plugins/documentation-examples/` so the next run reads the corrected shape, and amend the standard below if the change contradicts or extends a criterion.
- **One-off** — the change is specific to this document's subject. Nothing to fold.

When the standard and an example disagree, say so rather than silently picking one. The examples are the house style and usually win; the standard is what explains why.

If Phase 1 found no example for this document type, the accepted draft is the candidate for becoming one.

Report what was folded back and what was left as one-off.

---

## The Standard

Three tiers. Tier A decides what stays. Tier B fixes the prose that survives. Tier C fixes how it is presented.

### Tier A — Content fit

Run first. These cut whole blocks, and cutting a block makes its prose problems moot.

**A1. Doesn't serve the purpose.** The section is well-written and belongs in a different document. Measure it against the Phase 1 sentence, not against whether it is interesting.

**A2. Decision archaeology.** The reasoning trail written into a standing document — why an order changed, what the earlier plan was, what a past mistake taught. Replace with the current state stated plainly. The plan is the new order; it does not argue for it.

Flags: `Reordered [date]`, `Settled [date]`, `originally scoped`, `we decided`, `the reason this changed`, any narration of a prior version.

**A3. Content in the wrong document.** The test is ownership: if another document is the source of truth for a fact, this one links to it rather than restating it. A copy drifts the moment the owner changes — the plan that repeats a migration's open questions is wrong within a day of the migration answering them.

Appendices are usually this failure wearing a disguise. Content pushed to an appendix because it didn't fit is content that belongs in the document that owns it. If nothing in the body needs to link to it, it was never this document's to hold.

**A4. Reasoning argued more than once.** The same justification made in three sections. State it once, where the decision surfaces. Later sections state the consequence, not the argument.

### Tier B — Prose

**B1. Editorializing.** The writer's assessment leaking into the description. Flags: `elegant`, `powerful`, `robust`, `critical`, `significant`, `seamless`, `comprehensive`, `simply`, `just`, `cleanly`. Replace with the fact, unqualified.

**B2. Flourish.** Writing that performs. Aphorisms and epigrams where a plain statement belongs, sentence fragments used for rhythm, dramatic reveals, second person addressed to the reader.

The test: if a line could be lifted out and posted on its own, it is performing.

```
Before: That order asks reading to do a job only round-tripping can do.
After:  Schema problems surface on round-trip, not on review.

Before: The moment to watch for: turning it on and seeing your own migrated recipes for the first time.
After:  (deleted)
```

**B3. Narrative padding.** Sentences that connect sections rather than carry information. Flags: `Now that we've...`, `Let's turn to...`, `It's worth noting that...`, any sentence that leaves no gap when deleted. Replace with nothing — the heading is the transition.

**B4. Restating within a section.** The same claim made twice at different depths. Keep the one at the depth the reader needs to act on.

**B5. Session framing.** Written as a record of work rather than a statement of what is true. Flags: past tense about the system, first person plural, `as discussed`, `earlier we`. Replace with present tense.

Exception: session logs are records by definition. B5 does not apply to them.

**B6. Inflation.** Three sentences doing one sentence's work. Flags: `in order to`, `it is important to`, `serves to`, `generally speaking`, doubled adjectives. Hedge only where the uncertainty is real.

**B7. Overlong for the job.** A section that a reader skims needs to be skimmable. Repeated units — initiatives, goals, deferred items — get one sentence each unless a second is carrying a fact nothing else can. Two sentences is the ceiling, not the target.

The overflow rule: if a unit needs more than that, the extra content moves to the document that owns it, not to a longer paragraph and not to an appendix.

### Tier C — Form

**C1. Decoration that has stopped signalling.** Numbered emoji on already-numbered items, horizontal rules between sections that headings already separate, bold used for texture, emoji scattered through body text.

Top-level section headings carry an emoji by house convention — see the examples. That is a fixed pattern rather than decoration, and it is not what this criterion is about. Emoji below the top level, or emoji standing in for a word, usually are.

**C2. Heading weight.** A heading promises substance underneath. A heading carrying nothing but a sentence or two of prose does not deliver on that — use a bold label inline and let the content sit in a list. A heading is earned by what sits under it as a whole: a short paragraph plus a checklist is substance, a short paragraph alone is not.

**C3. Form doesn't match content.** Two problems, usually together.

Repeated units of the same shape get the same treatment throughout a document. A list of goals and a list of deferred items are the same kind of content — a label and a short reason — and should not be solved two different ways in the same file.

Numbering claims sequence. Use it only where order is real. Numbering an unordered set is a false claim, and it is worse when the document also numbers something genuinely ordered.

**C4. Status is shown more than one way.** Pick a single mechanism and let it carry. A checked list already says a thing is done — a completion line above it, a ✅ in the heading, and a dated log entry are three more ways of saying the same thing, and each one is a place the document can go stale.

**C5. Transitions.** Every heading with subheadings under it needs one, or the reader lands on a heading followed by another heading with nothing to orient them.

A transition is two sentences: what this section is, and why it matters. It never lists what's beneath it — that is what the headings and any overview block already do. It carries no background; background belongs in the summary if it belongs anywhere. If a section seems to need more than two sentences of introduction, the section needs restructuring, not a longer introduction.

**C6. Repeated units at uneven depth.** Units of the same kind get the same amount of prose. Four goals where two have descriptions and two don't, or ten initiatives where four are one sentence and six are two, read as unfinished even when every individual entry is good.

**C7. Prose doing a diagram's job.** Sequence, dependency, looping, and gates are structural facts. A fenced block that lays out the whole sequence — stages, order, what loops, what gates on what — communicates them in one look. Sentences describing the same relationships take longer to read and are harder to keep accurate.

**C8. Summary that isn't one.** A summary names what the document is, states the goal, says what follows, and stops. It is not the first section of the body, and it does not open with background the reader has to translate before it becomes useful.

---

## Notes

- This reviews writing, not correctness. A claim that appears factually wrong is raised as a question, never silently fixed.
- A document with no identifiable purpose after Phase 1 is raised as a question, not rewritten. It may not need to exist.
- Tier A does the work. A draft that only changed prose and form means either the document was already well-scoped or Phase 1 was too shallow.
- Tier C is where taste lives and where this SOP is least able to decide alone. Expect the most correction here, and fold each correction back into the standard.
- The examples are the house style; the standard explains it. When they conflict, the examples usually win and the standard gets amended in Phase 4.
- Expect Phase 3 to loop many times on form. The first draft usually gets content fit roughly right and form wrong, and each correction is worth more as a criterion than as a fix.
- Cutting is not finishing. A document is done when every remaining unit is the same shape and each one earns its place — not when the word count stops falling.
