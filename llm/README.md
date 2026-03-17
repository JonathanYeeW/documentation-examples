# LLM

Documents written specifically for LLM ingestion as part of a defined workflow. These are SOPs — procedures that an LLM follows when triggered by a specific phrase or condition. The document defines the trigger, the phases, and the gates.

These are not general reference docs. A human may author them, but the reader is a model. Write with that in mind: explicit triggers, sequential phases, clear gate conditions, and no ambiguity about what the model should do at each step.

## Audience

LLMs operating within a defined workflow. The individual running the workflow triggers the SOP; the model executes it.

## What belongs here

A document belongs in `llm/` if it defines a procedure for an LLM to follow — a sequence of steps the model works through when given a specific input or command. 

Documents that orient an LLM to a project or codebase belong in [`readmes/projects/`](../readmes/projects/) instead.

## Document Types

| Document | Purpose | Status |
|---|---|---|
| SOP (`sop`) | A procedure an LLM follows when triggered. Defines the trigger phrase, sequential phases, gate conditions, and expected output. | ✅ `sop_account-checkin.md` |
