# Project READMEs

A project README orients an LLM to a body of work. It is the first file a session reads, and it exists because a session starts with no memory — everything the model needs to be useful in the first minute has to be in this one document.

A human writes it, but the reader is a model. That is what separates it from a repo README: a developer can look around a codebase and infer, and a session cannot look around at all.

## When to Use One

**Every project directory gets one.** A project with no README is a project whose state lives in the last session's memory, which is to say nowhere.

- **Written for a cold start.** No prior context, no session history, no assumption that a name or an abbreviation is familiar. A term used without introduction is a term the reader has to guess at.
- **It holds current state, not history.** What is true now, what is active, what comes next. How it came to be true lives in the session log and the ticket workspace.
- **It grows organically.** A project README starts near-empty and accumulates through actual work. Writing one up front produces a document describing a plan rather than a project.
- **Not to be confused with a repo README**, which orients a developer to a codebase. A project may have both, and they share almost nothing. See [`../repos/`](../repos/).

## Conventions

**Four fixed sections, plus one named for the project.**

1. **Summary** — what the project is and what it is for
2. **What's Here** — the directory structure and what lives where
3. **Current State** — what is active, what is done, what is next
4. **To Pick Up the Work** — where to start, what to read first, any active blockers

The fifth section is named for what it holds — a post index, a list of people, the active ticket — whichever is most useful for this kind of project.

- **State a fact once, in the section that owns it.** A blocker named in Current State and again in To Pick Up the Work is a fact with two places to go stale.
- **Name the file when pointing at one.** "See the plan" costs a search; a path does not.
- **No editorializing.** The reader is deciding what to do next, and an assessment of how the work is going does not help them do it.

## Examples

| File | What It Covers |
|---|---|
| `rdm-project_sandwich-assembly.md` | A technical project — active tickets, accumulated domain knowledge, and a fifth section holding the current ticket |
| `rdm-project_pbj-blog.md` | A knowledge-work project — a post index as the fifth section, and state expressed as drafts rather than tickets |
