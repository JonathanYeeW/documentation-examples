# SOP: Make a Plan

**Created:** 2026-10-05
**Purpose:** Turn a conversation about a ticket into an implementation plan that a person or LLM arriving cold can read to understand how the ticket will be accomplished. The plan is written against [`docs/engineering/plans/README.md`](../docs/engineering/plans/README.md) and saved in the ticket workspace. The session that had the conversation runs this procedure.

---

## Trigger

Jonathan says: `"make a plan"`.

The ticket has already been talked through in this session, with the project and ticket loaded. If the ticket has no workspace yet, create one first.

Claude begins Phase 1.

---

## Phase 1 — Take Stock of the Conversation

Sort what the conversation produced by who settled it:

- **Decided by Jonathan** — anything he stated or agreed to.
- **Claude's readings** — anything Claude proposed or inferred that Jonathan didn't explicitly take up. A reading is not a decision, however long it has been in the conversation.
- **Still open** — questions raised and not answered, and build decisions the ticket leaves open. Tickets describe the product, so how it gets built is decided here.

Then fill gaps. Read the Linear ticket and its comments, the workspace README, or anything the ticket links to, if the session hasn't. If a plan in this project already covers the ticket, extend that plan rather than starting a second one.

No input needed at this step.

---

## Phase 2 — Load the Standards

Read the standards themselves, not only their indexes. Skip any file already read this session.

1. **The plans standard:** [`docs/engineering/plans/README.md`](../docs/engineering/plans/README.md), then both example plans in that directory.
2. **The coding standards:** `plugins/coding-standards/README.md`, then every file its "Applying a Standard" table routes to for the session types this ticket touches.
3. **List what was read**, with the rules from each that bear on this ticket, for the gate.

No input needed at this step.

---

## Phase 3 — Load the Repo's Documentation

Each repo keeps documentation about its own product in `docs/` at its root, laid out by `plugins/coding-standards/docs/repo-documentation.md`. Skip any file already read this session.

1. **Find the repo** from the project README's Project Context. If it isn't there, ask.
2. **Read `docs/README.md`**, then the documents it indexes that bear on this ticket.
3. **Read `docs/CONTRIBUTING.md`** and any `CONTRIBUTING.md` in the apps the ticket touches. A deviation recorded there overrides the cross-project standard.

If the repo has no `docs/`, say so at the gate.

No input needed at this step.

---

## Phase 4 — Investigate

Read the code the ticket will change, if the session hasn't. Then investigate each reading and open question from Phase 1 wherever its answer is: the code, the data, a log, a past ticket.

- **A question the code answers doesn't go to Jonathan.** A question only he can answer does.
- **Every claim the plan will rely on ends up decided, verified, or labelled as a guess**, by the plans standard's test.
- **Check that a plan is the right document.** Steps with no order that matters belong in the ticket. An approach still being argued needs an exploration first.

> ⏸️ **GATE → Jonathan.** Claude plays back only what is new since the conversation:
>
> - **What was read and found**, as bullets, each with the rule or fact that bears on the ticket. Anything that contradicts the conversation is flagged.
> - **Open decisions**, as bullets, each with Claude's recommendation.
> - **Claude's readings**, flagged as such.
> - **Proposed phases**, a few words each.
>
> Jonathan answers the decisions, corrects any reading, and confirms the phase order.

---

## Phase 5 — Write the Plan

Write the plan to the plans standard, at `projects/[project-name]/tickets/[eng-xxx]/plan_[descriptive-name].md`. When extending an existing plan, edit it in place.

- **Everything settled in the conversation and at the gate is written into the plan.** The reader wasn't there, so a plan that depends on the conversation can't be followed.
- **Test gates follow the testing standards** loaded in Phase 2.
- **Add the plan to the workspace README**, and remove any question the gate answered from its Open Questions.

> ⏸️ **GATE → Jonathan.** Review the plan. Nothing is built until he approves it.

---

## Notes

- **Questions are asked together at the gate**, not one at a time across Phases 1 to 4.
- **A standard that conflicts with the conversation** goes to the gate as a decision. The coding standards allow a deviation with a stated reason, and the plan records the reason.
- **A standard Jonathan points to after Phase 5** means rereading the plan against it and saying what it changes.
- **The plan is not committed.** It sits in the ticket workspace, outside the repo.
