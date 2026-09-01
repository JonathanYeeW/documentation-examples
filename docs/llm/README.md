# LLM

Documents written specifically for LLM ingestion as part of a defined workflow. Two types live here: SOPs, which an LLM runs when a phrase or condition triggers them, and plugins, which extend a session at a fixed point in it.

These are not general reference docs. A human may author them, but the reader is a model. Write with that in mind: explicit triggers, sequential phases, clear gate conditions, and no ambiguity about what the model should do at each step.

## Audience

LLMs operating within a defined workflow. The individual running the workflow triggers the SOP; the model executes it.

## What belongs here

A document belongs in `llm/` if it tells an LLM how to act — either a procedure it works through when given a specific input, or a constraint it holds for the length of a session. 

Documents that orient an LLM to a project or codebase belong in [`readmes/projects/`](../readmes/projects/) instead.

## Document Types

| Document | Purpose | Status |
|---|---|---|
| SOP (`sop`) | A procedure an LLM follows when triggered. Defines the trigger phrase, sequential phases, gate conditions, and expected output. | ✅ `sop_account-checkin.md`, `sop_write-pr-description.md` |
| Plugin (`plg`) | An extension loaded at a fixed point in a session — onboarding or offboarding. Either a standing constraint that holds for the whole session, or a conditional procedure that runs when its trigger is met. | ✅ `plg_machine-targeting.md`, `plg_shift-handoff.md` |

## Plugin Structure

Every plugin has three sections. A conditional plugin adds a fourth.

| Section | Required | What It Holds |
|---|---|---|
| `# 📋 Summary` | Always | Prose. What the plugin does, when it loads or runs, who reads it, and what they need from it |
| `# Trigger` | Conditional plugins only | The condition that causes it to run, and what happens when the condition is not met |
| Body | Always | Phases for a procedure, grouped rules for a standing constraint |
| `# 💬 Example` | Always | Inline. A plugin is self-contained and does not depend on a document outside itself |

Every heading carries a transition before its content — what the section is and why it matters, in no more than two sentences.

Derivation does not belong in a plugin. Why it was built this way lives in the ticket workspace that produced it.

`plg_machine-targeting.md` is the standing-constraint example. `plg_shift-handoff.md` is the conditional example, and it is the one to copy when the plugin has a trigger.
