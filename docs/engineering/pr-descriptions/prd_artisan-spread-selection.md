# [ENG-312] feat: choose a spread when ordering a sandwich

**Ticket:** ENG-312 · **PR:** #47

## Context

**Where this sits in the journey**

- A customer orders at the machine's screen: bread, then jelly, then a confirmation screen.
- After they confirm, the machine builds the sandwich and hands it over.

**Terms**

- **Spread** — what goes on the bread before the jelly: peanut butter, almond butter or sunflower butter.
- **Order** — one sandwich, from the first tap to the confirmation screen.
- **Out today** — a spread the machine has run out of. It's restocked each morning.

## Summary

At the machine, every sandwich comes with peanut butter, and a customer who wants something else, or can't eat peanuts, has no way to say so. This PR makes two changes:

1. **Choose a spread** — while ordering, a customer picks peanut, almond or sunflower butter after choosing their jelly.
2. **See what's out** — a customer can tell which spreads the machine has run out of before choosing, rather than after confirming.

## Change 1: Choose a spread

A customer who wants something other than peanut butter picks their spread as its own step, between jelly and confirmation.

```
Current:

  A customer orders a sandwich
  → Choose bread
  → Choose jelly
  → Confirm: "Sourdough · Grape"
  → The sandwich comes with peanut butter

New:

  A customer orders a sandwich
  → Choose bread
  → Choose jelly
  → Choose spread: Peanut, Almond, Sunflower          ← new
  → Confirm: "Sourdough · Grape · Almond butter"
  → The sandwich comes with almond butter
```

- Peanut butter is selected to start with, so a customer who always had it only taps Next.
- Going back to jelly and forward again keeps the spread they chose.
- The confirmation screen names the spread, so it can be changed before the machine starts.

## Change 2: See what's out

A customer sees that a spread is out on the spread step, before they pick it.

```
Current:

  The machine has run out of peanut butter
  → A customer orders a sandwich and confirms
  → The order stops partway: "Something went wrong"

New:

  The machine has run out of almond butter
  → A customer reaches the spread step
  → "Almond butter · Out today", and it can't be picked    ← new
  → They pick sunflower butter and confirm
```

- When peanut butter is out, the step starts on the first spread that isn't.
- When every spread is out, the welcome screen says so, and no order can start.
