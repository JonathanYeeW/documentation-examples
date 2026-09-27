# SOP: PR Description

**Created:** 2026-09-27
**Purpose:** Write a PR description with Jonathan, check it with a reader who wasn't in the session, and post it. The standard it is written against is [`docs/engineering/pr-descriptions/README.md`](../docs/engineering/pr-descriptions/README.md).

---

## Trigger

Jonathan drops a PR link in chat, or says `"write the PR description"`.

Begin Phase 1.

---

## The Principle

A PR description is the whiteboard defense, written down. Someone who knows the product but wasn't in the session reads it and can explain what changed for the user and why, without opening the code.

Everything below is how that gets tested before it is posted: Jonathan sets the altitude in his own words, the draft is written against the standard, and a reader with none of the session's context says where it loses them.

---

## Phase 1 — Questions

Load context first, without reporting it:

1. **The PR** — via GitHub MCP (`pull_request_read`, method: `get`) for the title, branches and size.
2. **What changed** — from the local clone when the session can reach one: `git log --oneline origin/[base]..origin/[head]` and `git diff --stat origin/[base]...origin/[head]`, reading individual diffs only where needed. Via GitHub MCP (`get_files`, then `get_diff`) when it cannot. A large PR's full diff does not fit in context; start from the commit list.
3. **What was decided** — each covered ticket's workspace README at `projects/[project]/tickets/[eng-xxx]/README.md`, and the session logs it lists. Linear only when no workspace exists.

Then ask Jonathan three questions, together:

1. **What are the changes, in your own words?** Asked before saying anything about the diff. His answer sets the altitude; the diff fills in the facts.
2. **Where does this sit in the customer journey?** The screen, and what the user is doing there. This anchors every other section, so it is settled first.
3. **What does someone need to know to follow it cold?** The terms, and the part of the product they'd need in their head.

From his answers, play back the Context section — the journey bullets and the term bank — as it would appear in the description.

> ⏸️ **GATE → Jonathan.** Correct the Context. Nothing is drafted until the journey and the terms are agreed.

---

## Phase 2 — Choose Who Drafts, and in What Style

The draft is always a file in the ticket workspace: `projects/[project]/tickets/[eng-xxx]/prd_[descriptive-name].md`. The cold read in Phase 4 needs a file to read, and GitHub only gets the final version.

Ask two questions, together:

1. **Who writes the first draft?** Claude, or Jonathan.
2. **Is there a writing style it should be in?** Name the styles in [`writing-styles/`](../writing-styles/) by folder, and accept any other Jonathan names. None is a valid answer.

> ⏸️ **GATE → Jonathan.** Choose who drafts, and the style if any.

---

## Phase 3 — Draft and Review

Write, or take, the draft against the standard in the type README: Context, Summary, then one Change section per item in the Summary. Before writing a line, read the examples in [`docs/engineering/pr-descriptions/`](../docs/engineering/pr-descriptions/) for the shape, and, when a style was chosen, that style's file and its examples for the language.

Deliver it with a note of three lines or fewer: what each Change section covers, and anything on the branch left out on purpose.

Jonathan reacts. Apply changes to the workspace file and re-deliver. Expect several passes, most of them about wording: the first draft usually has the content right and the language one level too close to the code.

> ⏸️ **GATE → Jonathan.** The draft reads right to him. Then the cold read.

---

## Phase 4 — Cold Read

Spawn a subagent with none of this session's context (`Agent`, `general-purpose`, in the foreground). It gets product knowledge a colleague would have, the description, and nothing else: no file paths, no ticket, no diff.

Fill in `[PRODUCT CONTEXT]` with five or six bullets on the product as a user knows it, written fresh from the project README, never from the draft. Fill in `[DESCRIPTION]` with the workspace file, verbatim.

```
You are reviewing a pull request description as a cold reader. Do not use any
tools: no file reads, no searches. Work only from what's in this message.

Who you are: someone at the company that makes [PRODUCT]. You know the product
as a user and a colleague, not the codebase. What you know about it:
[PRODUCT CONTEXT]

You were not part of the work session that produced this PR and you haven't
read the ticket. Skim this description the way a busy person reviews a PR,
then say what you understood and where you got lost.

What I want back, in this order:

1. Your one-paragraph explanation of what this PR changes and why, as if a
   colleague asked you at a whiteboard. Write what stuck after a normal skim.
2. Questions you'd ask the author, each quoting the line that raised it.
3. Gaps: things you'd expect this description to tell you that it doesn't.
4. Friction: anywhere the wording, jargon, symbols or structure made you stop
   and work something out, with the line quoted.
5. Anything you'd cut, because it didn't help you.

Be direct and specific. Don't praise it, don't rewrite it, and don't invent
problems. If you have nothing real under a heading, write "Nothing."

Here is the PR description:

---
[DESCRIPTION]
---
```

Report back to Jonathan in three groups:

- **Fix in the description.** A factual error, a term used before it is defined, a line two readers would take two ways.
- **Park.** A question about how the product should behave, not about what the description says. It goes in the ticket workspace's Open Questions, not the description.
- **Leave.** Anything the standard keeps out on purpose, with the rule that keeps it out.

The reader's whiteboard paragraph is the verdict. When it is right and the questions have turned from what the description means to how the product should behave, the description works.

> ⏸️ **GATE → Jonathan.** Choose which fixes to make. After substantial fixes, run the cold read again with a fresh subagent. A reader who saw the last draft is no longer cold.

---

## Phase 5 — Post

Post the workspace file as the description, and its first heading as the title, via GitHub MCP (`update_pull_request`). The workspace file stays as the record.

Confirm in chat with the PR link.

---

## Phase 6 — Fold Back

Two questions, asked together.

**Was the standard wrong?** Only when Phase 3 or Phase 4 showed it, not only this description. A correction Jonathan would make to any PR description is a correction to the type README; one about this change's subject is not. Propose each change to [`docs/engineering/pr-descriptions/README.md`](../docs/engineering/pr-descriptions/README.md), its examples, or this SOP.

**Is this description an example of its style?** Only when a style was chosen in Phase 2. Ask whether the posted description belongs in that style's examples, by the style's own rule for adding one. If it does, propose a PB&J rewrite of it for the style's `examples/`. The collection is published, so a real product never goes in.

> ⏸️ **GATE → Jonathan.** Accept or reject each proposal. Most runs have nothing for either question, and saying so is the whole of this phase.

---

## Notes

- **A branch covering several tickets** still gets one Summary. Group the Change sections by ticket only when the tickets touch different parts of the journey.
- **A PR with no user-facing change** — a refactor, a migration, test coverage — anchors in the journey of whoever it does change things for. A test PR's "user" is the engineer running the suite.
- **The cold reader's questions repeat across runs** — screenshots, undo, test steps. Answer them from the standard's "What stays out" rather than re-deciding them each time.
