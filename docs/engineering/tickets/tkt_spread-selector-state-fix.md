# [ENG-313] bug: keep the chosen spread when a customer goes back to change the bread

**Ticket:** ENG-313

## Context

**What the PB&J Machine is for**

Kids go hungry because the systems that feed them are slow, expensive and fragile. PB&J Machine Co. builds the software for a machine that can make any peanut butter and jelly sandwich on demand: a parent, a cafeteria worker or a food bank volunteer picks a bread, a spread and a jelly, and the machine makes it. The goal is a sandwich for any kid who needs one, with no training and no waste.

**Where this sits in the journey**

- A customer builds a sandwich one step at a time: bread, then spread, then jelly, then a review screen listing the sandwich before the machine starts.
- Back returns to the step before, so a customer can change an earlier choice.

**Terms**

- **Step** — one screen of building a sandwich, where the customer picks one thing: the bread, the spread or the jelly.

## Summary

A customer who goes back to change the bread loses the spread they already picked. The spread step comes back empty, and if they don't notice and carry on, the machine starts a sandwich with no spread and the order fails. After this ticket, going back to change one choice leaves every other choice as it was.

## Going back to change the bread

A customer who goes back from the review screen to change the bread finds the spread and jelly still picked on the way forward.

```
Current:

  Maya's sandwich: white, sunflower butter, grape, on the review screen
  → Back to the bread step → Wheat
  → The spread step: nothing picked
  → Pick sunflower butter again, or miss it and carry on
  → Missed: the order fails at the spread step

New:

  Maya's sandwich: white, sunflower butter, grape, on the review screen
  → Back to the bread step → Wheat
  → The spread step: sunflower butter still picked               ← new
  → The jelly step: grape still picked
  → The review screen: wheat, sunflower butter, grape
```

- It works the same going back to any step. Changing the spread keeps the bread and the jelly.
- A choice changes only when the customer picks something else on its step.
