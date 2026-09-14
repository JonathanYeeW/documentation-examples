# LLM

Documents written specifically for LLM ingestion as part of a defined workflow. Two types live here: SOPs, which an LLM runs when a phrase or condition triggers them, and plugins, which extend a session at a fixed point in it.

These are not general reference docs. A human may author them, but the reader is a model. Write with that in mind: explicit triggers, sequential phases, clear gate conditions, and no ambiguity about what the model should do at each step.

## Audience

LLMs operating within a defined workflow. The individual running the workflow triggers the SOP; the model executes it.

## What belongs here

A document belongs in `llm/` if it tells an LLM how to act — either a procedure it works through when given a specific input, or a constraint it holds for the length of a session.

Documents that orient an LLM to a project or codebase belong in [`readmes/projects/`](../readmes/projects/) instead.

## What's Here

| Directory | What It Covers |
|---|---|
| [`sops/`](sops/) | Procedures an LLM follows when triggered — trigger phrases, numbered phases, gate conditions, and a worked output |
| [`plugins/`](plugins/) | Extensions loaded at a fixed point in a session — standing constraints held throughout, or conditional procedures that run when their trigger fires |
