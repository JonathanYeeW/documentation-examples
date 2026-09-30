# [ENG-314] feat: mark a spread as out without removing it

**Ticket:** ENG-314

## Context

**What the PB&J Machine is for**

Kids go hungry because the systems that feed them are slow, expensive and fragile. PB&J Machine Co. builds the software for a machine that can make any peanut butter and jelly sandwich on demand: a parent, a cafeteria worker or a food bank volunteer picks a bread, a spread and a jelly, and the machine makes it. The goal is a sandwich for any kid who needs one, with no training and no waste.

**Where this sits in the journey**

- Staff decide which spreads the machine offers from **Spread Management** in the admin panel.
- When the supplier can't deliver a spread, the machine keeps offering it until staff do something.

**Terms**

- **Spread** — the middle layer of a sandwich, like peanut butter or sunflower butter.
- **Removed** — deleted from Spread Management. Bringing it back means setting it up again from scratch: name, portion and allergens.
- **Out** — still set up in Spread Management, but customers can't pick it. It's new with this ticket, and different from **Out today**, which the machine shows when a tub runs empty and clears at the morning reset.

## Summary

When a spread runs out for a while, staff can only remove it. Until staff remove it, customers can still order it, and once it's removed, bringing it back means setting it up again. After this ticket, staff can mark a spread as out and back again, and customers see it as out and can't pick it.

## Marking a spread out

A staff member who hears a spread is backordered marks it out in Spread Management, and marks it back when stock returns.

```
Current:

  The supplier says sunflower butter is backordered for two weeks
  → Spread Management: sunflower butter, with Remove
  → Remove it, or leave it offered
  → Two weeks later: Add spread → name, portion, allergens again

New:

  The supplier says sunflower butter is backordered for two weeks
  → Spread Management: sunflower butter, with Mark as out        ← new
  → Mark as out: it shows as Out in the list                     ← new
  → Two weeks later: Mark as back                                ← new
```

- Marking a spread out or back takes effect on the machine right away.
- Everything set up for the spread stays as it was while it's out.
- Remove still exists, for spreads the machine is dropping for good.

## What a customer sees

A customer choosing a spread sees one that's out marked "out", and can't pick it.

```
Current:

  Choosing a spread for Maya's sandwich
  → Sunflower butter is listed as normal
  → Pick it and start the machine
  → The order fails at the spread step

New:

  Choosing a spread for Maya's sandwich
  → Sunflower butter shows as "out", greyed                     ← new
  → Tapping it does nothing
```

- In Recent orders, a sandwich made with that spread shows the same way: listed, but can't be picked.
- A sandwich with that spread already on a review screen when it's marked out shows it as out, and the customer picks another before the machine starts.

## Open questions

- Should bread and jelly be markable as out too, or is this only for spreads?
- When a spread is marked out while a customer is on the review screen, when do they find out: straight away, or when they tap Start? Is Start blocked until they pick another?
- Should staff be able to set a return date, so a spread comes back on its own?
- Should customers see when a spread is expected back, or only that it's out?
