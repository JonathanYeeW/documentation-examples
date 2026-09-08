# Contributing to Documentation Examples

**Created:** 2026-09-07
**Applies to:** Every file in this collection
**Context:** The rules for adding an example, adding a document type, or changing one. Read this before creating a file here.

## Summary

This document covers what someone adding to the collection needs: what an example may reference, where a file goes, what it is named, how a new document type is introduced, and how a change gets accepted. What the examples themselves demonstrate, and which to read for a given task, is in [`README.md`](README.md).

## Audience

**Every reader of this collection is a stranger.** Usually an LLM session that has been pointed here and told to write a document of some type, occasionally a person deciding which type they need.

Neither arrives knowing the house style. That is the whole job of this collection — an example is read instead of a style guide, because a worked document communicates altitude, section order, and level of detail in a way a list of rules does not.

## Every Example Is PB&J

The PB&J Machine Company is the fictional setting for every example here: a team building the software for a magic peanut butter and jelly sandwich machine. Its domain is deliberately trivial — sandwiches, spreads, orders, shifts, operators, machines — so a reader's attention stays on the shape of the document rather than the business logic.

No example names a real project, a real repository, a real person, or a real customer. An example citing a real system stops reading as a shape and starts reading as that system's documentation, and the next session then has to judge which parts were general.

### The two settings

Examples that need a codebase are anchored to two fictional ones, so paths line up across files rather than each inventing its own.

**`pbj-api`** — a standalone Express backend.

**`pbj`** — a full pnpm monorepo: `packages/types` plus `apps/server`, `apps/mobile`, `apps/mcp-server`, `apps/cli`, and `apps/migrations`.

Neither exists. Read a repo name the way you read `SandwichProvider` — the shape is the point, the name is scaffolding.

## An Example Is the Template

There are no separate template files. The examples are what a new document is written against, and a template beside them is a second description of the same shape that drifts the first time one is edited.

A type with only one example is therefore doing double duty, which is why a type is worth a second example as soon as one exists that differs in a way the first cannot show — a different altitude, a different outcome, a structure the first had no reason to use.

## Naming

Files follow `{prefix}_{descriptive-name}.md`.

The prefix identifies the document type; the suffix describes the subject. A reader can tell what kind of document a file is without opening it. The prefix table in [`README.md`](README.md) is the register of recognized types, and a file whose prefix is not in it is either misnamed or a new type that skipped a step.

## Where a File Goes

`docs/` is organized into categories that mirror the layers of documentation in a software company. Inside a category, **a document type gets its own directory**.

| Level | Holds |
|---|---|
| `docs/<category>/` | One layer of company documentation — strategic, product, engineering, operations, readmes |
| `docs/<category>/<type>/` | One document type: a README describing the type, and every example of it |

**A type directory is named for the type in plural**, not for the prefix — `notebooks/`, not `nb/`. The prefix is how a file is named; the directory is how a reader finds it.

**Placing a file is a claim about which layer it belongs to.** A document that could sit in two categories belongs in the one whose audience it is written for, not the one whose subject it covers.

## What a Type README Holds

The README is the reason a type gets a directory rather than a row in a table. It carries what an example cannot show by being read: when to reach for this type, what separates it from its neighbours, and the conventions that hold across every instance of it.

Four sections, in this order:

| Section | Holds |
|---|---|
| Opening | What this document type is, in one or two paragraphs. No heading — it sits directly under the title |
| When to use one | The condition that makes this type the right choice, and the types it is most often confused with |
| Conventions | The rules that hold for every document of this type — structure, what goes in, what never does |
| Examples | A table naming each example and what it covers that the others do not |

`docs/engineering/notebooks/README.md` is the reference shape.

**The opening states what the type is, not what the directory contains.** A reader who has landed here already knows there are examples below.

**Conventions are rules, not description.** A convention a reader could have inferred from the example does not need stating; one they would get wrong without being told does.

**A type README is not a wiki README**, despite both sitting at the front of a directory. A wiki README fronts a directory of standards it does not own, so it routes and holds nothing — [`docs/readmes/wikis/README.md`](docs/readmes/wikis/README.md) is explicit that a rule lives in the child that proves it. A type README owns the standard for its type; its children are examples, and an example cannot state a rule. That inverts the constraint: here the rules go in the README, because there is nowhere below it for them to live.

**The category README is the wiki README of this collection.** `docs/<category>/README.md` routes to its type directories and describes what each covers. It does not carry the conventions of any type.

## Adding a New Type

Five steps, in order. The order matters — the example is what the other four describe.

1. **Write the example first.** A type with no worked instance is a shape nobody has tested. Write it in PB&J and get it accepted before anything else is created.
2. **Create the type directory** and move the example into it.
3. **Write the type README** against the accepted example.
4. **Add the prefix** to the table in [`README.md`](README.md).
5. **Add the type** to its category's README, and to the category table in [`README.md`](README.md) if the category was empty.

A type that stops at step 1 is an example without a home, and a type that starts at step 3 is a README describing a document nobody has written.

## Amending and Retiring

**When an example changes, the old version does not stay.** Update it in place. Why it changed lives in the session log or the ticket workspace that produced the change.

**A correction that only fixes one example gets made again.** When a review corrects something that would apply to any document of that type, the type README gets the rule and the other examples of that type get the fix.

**Retiring a type means deleting its directory and removing its rows.** A type left in place with a note saying it is superseded is a type some session will still read and follow.

## Review and Acceptance

**A change here is accepted by Jonathan, not merged on its own.** These examples are what every future document is written against, so a wrong shape propagates further than a wrong document.

**A substantial rewrite runs [`procedures/sop_document-review.md`](procedures/sop_document-review.md).** It gates on purpose and on the reference example before drafting, and on acceptance before replacing the original. Its Phase 4 is what folds an accepted correction back into the type it came from.

**Writing a document from scratch has no input document**, so the review SOP's Phase 2 writes the real file rather than a draft beside an original. Every other phase runs unchanged.

## Commit Messages

Conventional commit prefixes, all lowercase — `feat:`, `bug:`, `refactor:`, `build:`, `docs:`. Changes here are `docs:`.
