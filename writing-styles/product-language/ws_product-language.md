# Writing Style — Product Language

**Created:** 2026-09-27
**Who reads it:** anyone who knows the product: an engineer, a product person, someone reading months later
**When to use it:** explaining a change to someone who wasn't there. PR descriptions first, and anything else written about the product rather than the code

## Summary

Product language describes a change from the point of view of the person using the product: where they are, what they're trying to do, and what they see. A reader who knows the product but has never seen the code should be able to explain the change and defend it afterwards. The examples are the style; this file only says what they have in common.

**What it does**

- **Starts from the problem as the user meets it**, not from the system: "On the review screen, changing the bread on one sandwich means going back through every step for it."
- **Names each change by what the user can now do**, with the user as the subject: "a customer can copy a sandwich to another person in the same order."
- **Says what the user sees.** "It shows as out today and can't be picked", not "it is disabled when inventory is zero".
- **Uses the product's real screen names and labels**, exactly as they appear: "Remove sandwich", not "Remove".
- **Defines any word with two meanings** before using it, like a sandwich from the menu and a custom sandwich.

**What it leaves out**

Anything a product reader wouldn't ask about: function, file and test names, how it was tested, small changes that rode along, and other tickets' dependencies. The code is where those live.

## Examples

Each example is a whole piece written in this style, in [`examples/`](examples/).

| File | What it shows |
|---|---|
| [`prd_family-order-change-and-copy.md`](examples/prd_family-order-change-and-copy.md) | A PR description with three changes: journey and terms before anything else, the problem in the Summary, and each change told as a current and new flow with the answers to what a reader would ask |

## Adding an Example

When a piece written in this style is accepted, ask whether it's worth adding. It is when it shows the style somewhere the examples don't yet: a different kind of document, a different kind of reader, or a problem the others didn't have.

Rewrite it as PB&J before adding it. This collection is published, so an example never names a real product, project or person. Keep its shape, its length and its decisions, and change only the subject.

Add it whole to `examples/`, named with its document type's prefix, and give it a row in the table above saying what it shows. When a newer example shows the same thing better, it replaces the older one rather than joining it.
