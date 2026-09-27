# [ENG-318] feat: restock a spread from the operator screen

**Ticket:** ENG-318 · **PR:** #55

## Context

**Where this sits in the journey**

- An operator runs a shift at the machine: refilling ingredients, clearing jams and handing over to the next operator.
- They work from the operator screen on the back of the machine, which customers never see.
- Customers pick a spread while ordering. A spread the machine has run out of shows as "Out today" and can't be picked.

**Terms**

- **Operator** — the person looking after a machine during a shift. Not a customer.
- **Out today** — a spread the machine has run out of. Customers can see it but can't pick it.
- **Restock** — refilling a spread's tub, which makes it available to customers again.

## Summary

When an operator refills a spread mid-shift, it stays "Out today" for customers until the morning reset, so a full tub sits unused for the rest of the day. This PR makes two changes:

1. **Restock a spread** — an operator can mark a spread refilled from the operator screen, and customers can pick it again straight away.
2. **See it at handover** — the next operator sees which spreads were restocked, and when, in the shift handover.

## Change 1: Restock a spread

An operator who has refilled a spread marks it restocked on the operator screen, and the next customer can pick it.

```
Current:

  Almond butter runs out at 11am
  → The operator refills the tub
  → Customers still see "Almond butter · Out today"
  → It comes back at the morning reset, tomorrow

New:

  Almond butter runs out at 11am
  → The operator refills the tub
  → Operator screen → Spreads → Almond butter → Restocked    ← new
  → The next customer can pick almond butter
```

- Only a spread that's out shows Restocked. An operator can't mark a full tub restocked.
- A customer partway through an order when it's restocked sees it the next time they reach the spread step.
- The morning reset still restocks everything, whether or not an operator did it during the day.

## Change 2: See it at handover

The operator taking over a shift sees what was restocked during the one before.

```
Current:

  An operator hands over at 2pm
  → The handover lists jams cleared and orders stopped
  → Nothing says almond butter was refilled at 11am

New:

  An operator hands over at 2pm
  → The handover lists jams cleared and orders stopped
  → "Restocked: Almond butter, 11:04am"                     ← new
```

- A spread restocked twice in one shift is listed once per restock, with each time.
- The handover lists restocks by operators only. The morning reset isn't a restock anyone did, so it isn't listed.
