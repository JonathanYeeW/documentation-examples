# PR Descriptions

A PR description lets anyone who knows the product understand, by skimming, what changed for the user and why, without reading the code. The test is the whiteboard: after reading it, someone who wasn't there can explain the change and defend its decisions.

It sits on top of the diff, and the diff already says how. The description says what the user can do now that they couldn't before, and what they'll see.

The procedure for writing one — the questions, the draft, the cold read, the post — is [`procedures/sop_pr-description.md`](../../../procedures/sop_pr-description.md).

## When to Write One

**Every PR gets one.** Its reader is anyone at the company who knows the product: an engineer reviewing it, a product person, or someone reading it months later. Written for the least technical of them, it still works for the rest.

- **Not release notes.** Release notes tell people outside the team what shipped across several PRs. A PR description covers one change set, for the people building the product. See [`../shipping/`](../shipping/).
- **Not a ticket.** A ticket describes a problem before anyone has solved it. A PR description describes what the solution does.
- **Not a plan.** A plan orders work that hasn't happened. A PR description reports work that has.

## Conventions

### Structure

| Section | Holds |
|---|---|
| Title | `[TICKET-ID] type: summary`, conventional commit prefix, all lowercase. The first heading repeats it exactly |
| Title block | `Ticket` and `PR` on one line |
| `## Context` | Where this sits in the customer journey, then a term bank |
| `## Summary` | The problem as the user meets it, then a numbered list of the changes |
| `## Change N: <name>` | One section per change in the Summary, in the same order and under the same name |

Nothing else. A section that doesn't fit one of these belongs in the ticket workspace.

### Context

**Context comes before any change.** It is what the reader needs in their head to follow the rest, and nothing about the changes themselves.

- **Where this sits in the journey** is a bolded label and a few bullets: the screen, what the user is doing there, and how they got there. Every other section is read against it.
- **Terms** is a bolded label and a bullet per term, each a bold word and a one-line definition. A term earns a place when the description uses it in a way a product reader could misread, most often a word with two meanings. When two terms are easy to confuse, one example sentence saying they differ is worth more than a longer definition.

### Summary

**The Summary opens with the problem, not the product.** One or two sentences on what a user couldn't do, or had to do the long way, as they'd describe it: "at the machine, every sandwich comes with peanut butter."

Then `This PR makes N changes:` and a numbered list. Each item is a bolded name for what the user can now do, a dash, and one sentence with the user as its subject.

### Changes

**One section per numbered item**, titled `Change N: <the name from the Summary>`.

- **An opener of one or two sentences**, with the user as the subject, saying what they can do now.
- **A current and new block**: one fenced block, `Current:` and then `New:`, each a sequence of single actions starting from the same situation. Mark the line that's new with `← new`. The two blocks should differ only where the change is.
- **Bullets for what a reader would ask.** Each answers a question a product reader would actually raise: what happens when it collides with something already there, what the user sees at the edges, what stays the same. A bullet that answers a question nobody would ask comes out.

### Product language

**Write it in the terms of the product, not the code.**

- **Start from the problem as the user meets it**, not from the system.
- **Name each change by what the user can now do**, with the user as the subject.
- **Say what the user sees.** "That meal disappears from the day", not "the meal is removed".
- **Use the app's real screen names and labels**, exactly as they appear.
- **Define any word with two meanings** in the term bank before it is used.

### What stays out

**Anything a product reader wouldn't ask about.** The diff and the file list show an engineer where to look.

- Function, file and test names.
- How it was tested. The checks run on the PR, and the reader can't act on them.
- Smaller changes that ride along on the branch. Whoever reads the code will find them.
- Dependencies on other tickets. They go on the ticket they affect.
- Screenshots, unless the author adds them.

## Examples

| File | What It Covers |
|---|---|
| [`prd_artisan-spread-selection.md`](prd_artisan-spread-selection.md) | Two changes to one flow, each with its own current and new block. Change 2's current block has no spread step at all, since Change 1 adds it, so "current" is the failure a user hits today |
| [`prd_jelly-follows-bread-size.md`](prd_jelly-follows-bread-size.md) | A fix with one change and nothing different on screen. The only visible difference is the sandwich, so the blocks show the outcome rather than a new step, and the Summary names the incident it closes |
| [`prd_restock-from-the-operator-screen.md`](prd_restock-from-the-operator-screen.md) | A change for someone who isn't the customer. The journey is an operator's shift, and the term bank says who an operator is before anything else, because a reader would assume "user" means customer |
