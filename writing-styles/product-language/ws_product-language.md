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

Each example is a whole piece written in this style, kept in `examples/` beside this file. Git ignores every `examples/` folder, so each person keeps their own, taken from their own projects, and none is published. Read whatever is there before writing in this style.

A fresh clone has none. This file is then the whole description of the style.

## Adding an Example

When a piece written in this style is accepted, ask whether it's worth adding. It is when it shows the style somewhere the examples don't yet: a different kind of document, a different kind of reader, or a problem the others didn't have.

Copy it whole into `examples/`, named with its document type's prefix. It stays as written, real product and all, since it never leaves the machine. When a newer example shows the same thing better, it replaces the older one rather than joining it.
