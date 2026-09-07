# Contributing Docs

A contributing doc is the second front door of a code repository. Where the repo README tells a developer what the project is and how to run it, this one tells them how to work inside it — the architecture they are about to change, how it is laid out, how to run it locally, and what a change has to satisfy before it lands.

It lives at `CONTRIBUTING.md` in the repo root and is written for a developer who has already decided to make a change. That is what separates it from the README: the README's reader is deciding whether this repo is the one they want, and this reader has stopped deciding.

## When to Use One

**A repo gets a contributing doc when it has conventions a newcomer would otherwise violate.**

A repo whose structure is obvious from its directory listing does not need one. A repo with a layered command structure, a shared `lib/` that must stay domain-free, or a test suite that expects a simulator to be running does.

- **The test is whether a correct-looking change can be wrong.** If someone can put a file in a reasonable place and break a rule nobody stated, write one.
- **It is not a duplicate of the README.** Setup instructions belong in the README. Architecture, layout rules, and testing expectations belong here.
- **It is not a coding standard.** A standard describes how every repo works; this describes how one repo works. When a rule here would hold for any repo of its kind, it belongs in the standard instead.
- **The same type covers documentation collections.** A wiki or an examples collection has contributors too, and the rules for adding to it are the same kind of document.

## Conventions

- **Open with what the document covers**, in one or two sentences, and then the first section. No welcome, no restating the repo's purpose.
- **Architecture comes first**, because everything else in the document assumes it.
- **A directory tree is annotated.** A tree with no comments is a listing the reader could have produced themselves; the value is the one-line note on what each directory owns.
- **State the layering rule out loud.** The rule that a shared directory holds nothing domain-specific is the kind of thing a tree implies and a sentence enforces.
- **Local development is commands, not prose.** Each fenced block does one job, with a line above it saying when to reach for it.
- **Testing says what a new change owes.** A section that only explains how to run the suite has not said what a contributor has to write.
- **No history.** Why the architecture is shaped this way lives in the ticket that shaped it.

## Examples

| File | What It Covers |
|---|---|
| `ctb_cli-pbj.md` | A CLI repo — two-phase command routing, a command/handler/service layout, a `lib/` that owns no domain knowledge, a machine simulator for local runs, and what tests a new command owes |
