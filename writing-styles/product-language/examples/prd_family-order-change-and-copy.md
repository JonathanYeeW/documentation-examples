# [ENG-347] feat: change bread and copy from the ••• menus, and recent orders include custom sandwiches

**Ticket:** ENG-347 · **PR:** #61

## Context

**Where this sits in the journey**

- A customer orders for their family at the machine: a sandwich for each person, then a review screen listing every sandwich before the machine starts.
- To add a sandwich, they either build it step by step or pick one from Recent orders.

**Terms**

- **Sandwich** — one bread, one spread and one jelly, for one person.
- **Custom sandwich** — a sandwich built step by step rather than picked from the menu, like sourdough with sunflower butter and apricot.
- **Family order** — every sandwich for one visit, reviewed together before the machine starts.
- **Who it's for** — the name on each sandwich in a family order. The machine prints it on the wrapper.
- **`•••` menu** — the menu beside each sandwich on the review screen. Before this PR it offered only Remove sandwich.

## Summary

On the review screen, changing the bread on one sandwich means going back through every step for it, and there's no easy way to make the same sandwich for a second person. This PR makes three changes:

1. **Change the bread** — while reviewing, a customer can switch a sandwich to another bread from its `•••` menu.
2. **Copy a sandwich** — a customer can copy a sandwich to another person in the same order, from the same menu.
3. **Recent orders lists sandwiches** — when adding a sandwich, a customer sees the ones they ordered recently, including custom ones.

## Change 1: Change the bread

A customer who picked the wrong bread can tap the sandwich's `•••` and pick Change bread, instead of stepping back through the order.

```
Current:

  Maya's sandwich is on white, but should be on wheat
  → Tap ••• on the sandwich: Remove sandwich
  → Back → Back → Back to the bread step → Wheat
  → Forward through spread and jelly again

New:

  Maya's sandwich is on white, but should be on wheat
  → Tap ••• on the sandwich: Change bread, Copy, Remove sandwich   ← new
  → Change bread → Wheat
```

- The spread, the jelly and who it's for stay the same. To change those, the customer still goes back through the steps.
- A bread the machine has run out of shows as "out today" and can't be picked.

## Change 2: Copy a sandwich

A customer who wants the same sandwich for someone else can tap its `•••`, pick Copy, then choose who it's for.

```
Current:

  Maya and Leo both want wheat, peanut butter and grape
  → Build Maya's sandwich
  → Add a sandwich, build the same one again for Leo

New:

  Maya and Leo both want wheat, peanut butter and grape
  → Build Maya's sandwich
  → Tap ••• on it: Change bread, Copy, Remove sandwich             ← new
  → Copy → Leo
```

- The copy is its own sandwich. Changing Leo's bread afterwards doesn't change Maya's.
- Copying to someone who already has a sandwich gives them a second one.
- The names offered are the people already in the order, plus Someone new.

## Change 3: Recent orders lists sandwiches

When a customer adds a sandwich and opens Recent orders, they see the sandwiches they ordered, not the menu items behind them, so a custom sandwich shows up too.

```
Current:

  Leo had a custom sandwich last Tuesday: sourdough, sunflower, apricot
  → Add a sandwich → Recent orders
  → Menu sandwiches ordered recently
  → Leo's sandwich isn't listed

New:

  Leo had a custom sandwich last Tuesday: sourdough, sunflower, apricot
  → Add a sandwich → Recent orders
  → "Sourdough, sunflower, apricot · Leo · Tuesday"             ← now listed
  → Tap it → it's added for Leo
```

- The customer sees sandwiches from the last two weeks.
- The same sandwich ordered more than once shows once, from its latest order.
- A sandwich whose bread, spread or jelly is out today shows, but can't be picked.
