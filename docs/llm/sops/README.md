# SOPs

A standard operating procedure is a document an LLM executes rather than reads. It defines a trigger, a sequence of phases, the points at which it stops and waits for a human, and what it produces. The model works through it top to bottom; the individual who triggered it supplies judgment at the gates.

An SOP exists because a procedure that lives in someone's head is run differently every time. Writing it down makes the output consistent regardless of which session runs it or how much context that session arrived with.

## When to use one

**Write an SOP when the same multi-step procedure gets run repeatedly and its output should look the same every time.**

- **The procedure has a trigger.** Something specific starts it — a phrase, a link dropped in chat, a condition being met. A procedure with no trigger is a plan, not an SOP.
- **The steps have an order that matters.** If the steps could be done in any sequence, a checklist in a README serves better.
- **A human needs to intervene partway.** Gates are the point. A procedure that runs unattended end to end is a script, and should be written as one.
- **Not a plugin.** A plugin loads at a fixed point in a session and either holds a constraint for the whole session or runs when its trigger fires. An SOP is invoked deliberately to produce something. See [`../plugins/`](../plugins/).
- **Not a knowledge base article.** A `kb-op` explains how a system works. An SOP tells a model what to do. If the reader finishes and knows something rather than having produced something, it is a `kb-op`.

## Conventions

### Structure

Sections in this order:

| Section | Required | What It Holds |
|---|---|---|
| Title block | Always | `Purpose` at minimum — what the procedure produces, in one or two sentences. `Created` where the date matters |
| `## Trigger` | Always | The exact phrase or condition that starts it, and what the model does first |
| `## Phase N — <name>` | Always | The work, in order. One phase per meaningful unit |
| `## Notes` | Optional | Edge cases, assumptions about context, what to do when the procedure does not quite fit |
| `## Example` | Always | A finished artifact produced by the procedure |

**Phases are numbered and named.** `## Phase 2 — Build the Doc`, not `## Step 2`. The name is what a session uses to say where it is.

**The trigger is written as the words a person actually says**, in backticks. A trigger described rather than quoted gets paraphrased, and then the SOP does not fire.

### Gates

**A gate is a blockquote, and it names who it is waiting on.**

```
> ⏸️ **GATE → Account Manager.** React to the synthesis. Correct anything
> stale, add anything the CRM missed, or just say "looks good."
```

**A gate states what the human is being asked to do**, not merely that the model should pause. "Confirm before continuing" gives the reader nothing to confirm against.

**Gate where being wrong is expensive to undo.** After loading context and before acting on it; before anything is written to a shared surface; before a destructive step. A procedure with a gate between every phase is a conversation with extra formatting.

### Writing for a model

**Name the tool and the method.** `via GitHub MCP (pull_request_read, method: get_files)` rather than "fetch the PR files." A session that has to guess which tool will guess differently each time.

**Specify the output shape, not just the topic.** Section by section: how long, what form, what it should read like. `2–3 sentences of prose. Not bullets.` is a specification; "write a summary" is not.

**Say where output goes**, with the path template written out, when the procedure saves something.

**State what does not need input.** `No input needed at this step.` stops a model from inventing a gate that was not there.

### The Example

**Every SOP ends with a worked output**, separated by a horizontal rule and introduced by a line naming what it is an example of.

The example is the artifact the procedure produces, not a transcript of the procedure running. A reader comparing their output against it should be comparing like for like.

## Examples

| File | What It Covers |
|---|---|
| [`sop_account-checkin.md`](sop_account-checkin.md) | A procedure that produces a document and saves it — three phases, one gate after context loading, detailed per-section output specs, and a path template |
| [`sop_write-pr-description.md`](sop_write-pr-description.md) | A procedure that writes to an external system — named MCP tools and methods, a `Notes` section for when the procedure does not quite fit, and a gate placed after the write rather than before it |
