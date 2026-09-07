# Repo READMEs

A repo README is the entry point for a codebase. One per repository, at the root, written for a developer who has just arrived — either for the first time or after long enough away that nothing is in their head.

It answers three questions and stops: what is this, how do I run it, and where do I go next. Anything past that belongs in the contributing doc, which is the document for a developer who has already decided to make a change.

## When to Use One

**Every repository gets one.** There is no threshold — a repo with no README is a repo whose setup lives in somebody's memory.

- **The reader is deciding, not building.** They want to know whether this is the repo they need and what it takes to get it running. A layered explanation of the architecture answers a question they have not asked yet.
- **Setup has to actually work.** A README's commands are the one part of a repo that a newcomer runs before they can verify anything, so an out-of-date install step costs more than an out-of-date paragraph.
- **The flavour follows the repo type.** An API README leads with endpoints and environment variables; a mobile one leads with simulators and build profiles. The shape is shared and the emphasis is not.
- **Not to be confused with a project README**, which orients an LLM to a body of work rather than a developer to a codebase. See [`../projects/`](../projects/).

## Conventions

- **Open with one or two sentences on what the repo is**, then go straight to setup. No badges-and-tagline preamble.
- **Prerequisites are versions, not names.** "Node 24" is actionable; "Node" is not.
- **Every command is copy-pasteable** and appears in the order a newcomer runs it.
- **Environment variables are listed with what each one is for**, and never with a real value.
- **Point at the contributing doc rather than absorbing it.** Architecture and testing expectations have their own document.
- **No history and no roadmap.** What the repo is going to become belongs in a plan.

## Examples

| File | What It Covers |
|---|---|
| `rdm-repo_api-pbj.md` | A backend service — endpoints, environment variables, and running against a local database |
| `rdm-repo_cli-pbj.md` | A CLI — global install, the command list, and running against a simulator |
| `rdm-repo_mcp-pbj.md` | An MCP server — the tools it exposes and how to wire it into a client |
| `rdm-repo_mobile-pbj.md` | A mobile app — simulators, build profiles, and the two data-source modes |
| `rdm-repo_web-pbj.md` | A web frontend — dev server, build, and the API it points at |
