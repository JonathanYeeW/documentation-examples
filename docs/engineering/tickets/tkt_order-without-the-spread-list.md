# [ENG-315] feat: let an order go through when the machine can't reach the spread list

**Ticket:** ENG-315

## Context

**What the PB&J Machine is for**

Kids go hungry because the systems that feed them are slow, expensive and fragile. PB&J Machine Co. builds the software for a machine that can make any peanut butter and jelly sandwich on demand: a parent, a cafeteria worker or a food bank volunteer picks a bread, a spread and a jelly, and the machine makes it. The goal is a sandwich for any kid who needs one, with no training and no waste.

**Where this sits in the journey**

- A customer picks a bread, a spread and a jelly for each sandwich, checks the order on the review screen and taps Start.
- Before it starts, the machine checks each spread against the spread list, to make sure it's still offered and to get its portion and allergens.
- The spread list is kept in one place for every machine. Staff edit it in **Spread Management**.
- It changes rarely: a new spread, a changed portion, a spread removed.

**Terms**

- **Spread list** — every spread the machines can make, with its portion and allergens. One list, shared by every machine.
- **Last copy** — the spread list as the machine last saw it. Every machine keeps one.

## Summary

When the spread list can't be reached, even for a minute while it's being updated, every order started in that minute fails, and the customer sees an error. The machine gives up on the check even though the list rarely changes and it already holds a recent copy. After this ticket, the machine checks against its last copy when the list can't be reached, the order goes through, and staff can see that it happened.

## Starting an order while the list can't be reached

A customer who taps Start while the spread list can't be reached gets their sandwich, checked against the machine's last copy.

```
Current:

  Leo's family order: three sandwiches, on the review screen
  → Tap Start
  → The spread list can't be reached
  → "Something went wrong. Please try again."
  → Leo tries again, or leaves without the sandwiches

New:

  Leo's family order: three sandwiches, on the review screen
  → Tap Start
  → The spread list can't be reached
  → The machine checks the spreads against its last copy          ← new
  → The machine starts
```

- The customer sees nothing different. The order starts as it would any other time.
- When the list can be reached, the machine checks against it as it does today, and updates its last copy.

## What staff see

Staff can see each time a machine started an order from its last copy, so they know when the spread list couldn't be reached and for how long.

- An order started from the last copy is marked as one, with the time.

## Open questions

- How old can the last copy be before the machine stops trusting it and refuses the order?
- A spread that was removed or marked out just before the list went unreachable is still in the last copy. Is starting an order with it acceptable?
- The same goes for a spread whose allergens just changed. Should an allergen change make the machine refuse the order until it can reach the list?
- Where do staff see orders started from the last copy: in Spread Management, or somewhere for the whole machine?
