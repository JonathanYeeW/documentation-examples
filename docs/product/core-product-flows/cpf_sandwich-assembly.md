# Core Product Flow — Order to Sandwich

**Created:** 2026-03-01
**Last Updated:** 2026-10-20
**Context:** From someone walking up to the machine to a sandwich in their hand.

## Summary

The customer builds an order on the screen, the machine builds a sandwich from that order, and the sandwich comes back to the customer, usually in under two minutes. Once the customer taps **Make My Sandwich**, the machine never asks them anything again. Everything after that is read from the order and from the Spread Registry, the list of spreads the machine knows and how each one is handled. Quality isn't one inspection at the end. Three gates sit along the build, plus a mix check on spreads that need stirring, each at the earliest step that can see its problem, because a failure costs more with every step already taken. Read this before changing any part of the path, since every feature touches some step of it.

## Flow at a Glance

```
PHASE 1: ORDER (the customer, 15–20 seconds)

  1. Customer taps the welcome screen
  2. Customer picks bread, and toasting if offered
  3. Customer picks a spread
  4. Customer picks a jelly
  5. Customer checks the order → Make My Sandwich
  6. Screen hands the order to the machine

        ~~~~~~~~ handoff to the machine: the order ~~~~~~~~

PHASE 2: ASSEMBLY (the machine, 60–90 seconds)

  7.  Machine pulls the ingredients      — ingredient gate  ✗ order fails
  8.  Machine readies the spread         — mixed?           ✗ order fails
  9.  Machine slices, toasts, cools the bread
  10. Machine spreads, checks, combines  — coverage gate    ✗ one touch-up,
                                                              then order fails
  11. Machine checks, photographs        — final gate       ✗ order fails
      → order ready

      → A failed order tells the customer why and offers a new one

        ~~~~~~~~ handoff to the customer: the sandwich ~~~~~~~~

PHASE 3: DELIVERY (5 seconds)

  12. Machine tells the customer it's ready
  13. Customer takes the sandwich, welcome screen returns
  14. Machine logs the order
```

---

## Phase 1: Order

The customer's phase. A guided wizard asks for one choice per screen, and nothing reaches the machine until step 6.

**Step 1: The customer taps the welcome screen.** The machine waits on a welcome screen until someone touches it.

- Tapping anywhere starts an order.
- There's no login and no account. The machine serves whoever is standing in front of it.

**Step 2: The customer picks bread.** The first of three choices, each shown as cards with a short description.

- A toasting toggle appears under the cards when the machine can toast.
- The toggle starts off.

**Step 3: The customer picks a spread.** The same cards as bread, and the only place the Spread Registry shows through to the customer.

- Each card carries a note from the registry, such as "This spread is natural and will be stirred fresh" for almond butter.
- **Why the notes exist** — they warn the customer about a longer wait without explaining how the machine works.

**Step 4: The customer picks a jelly.** The same cards with no notes, because no jelly changes how long the customer waits.

**Step 5: The customer checks the order.** The last moment before any ingredient moves.

- One screen shows every choice, with **Make My Sandwich**.
- Tapping a choice goes back to its screen to change it.

**Step 6: The screen hands the order to the machine.** The order says three things, such as "white bread, toasted; almond butter; grape jelly", and nothing about who placed it. From here the machine works from the order alone.

- The screen switches to the **progress screen**, showing which phase the machine is in.
- The customer waits there through Phase 2 and makes no further choices.

---

## Phase 2: Assembly

The machine's phase. Each gate is named by the question it answers, and what each one checks and costs is in `wiki_quality-model.md`.

**Step 7: The machine pulls the ingredients.** The order becomes bread, spread and jelly staged at the prep station, and the ingredient gate asks "Do we have what we need, and is it good?"

- Every ingredient must be in stock and in date.
- A failure here costs nothing, because no ingredient has been used yet. This is the cheapest place to stop.

**Step 8: The machine readies the spread.** Decided by the Spread Registry, not by the order. The order says almond butter; the registry says almond butter separates in the jar and has to be stirred.

- Natural spreads are stirred, up to 3 times. One that still hasn't mixed fails the order.
- Refrigerated spreads are warmed to room temperature, which the registry calls tempering. Warming that runs past its limit, set in `cpf_assembly-pipeline.md`, carries on with a warning rather than failing the order.
- Standard peanut butter skips this step.
- The spread is readied before the bread is touched, so a spread that won't mix fails the order with no bread used.

**Step 9: The machine slices, toasts and cools the bread.** Done to the order: sliced if needed, toasted if the customer asked, laid out for assembly.

- Toasted bread cools for 30 seconds before any spread goes on.
- **INC-042** — without the cool-down, spread melts into the bread and the sandwich falls apart on the first bite.

**Step 10: The machine assembles the sandwich.** The same fixed sequence for every order, with the coverage gate asking "Did the spread and jelly go on properly?" before the slices close.

- Spread goes on one slice, jelly on the other.
- A bare spot larger than 1cm² gets one touch-up. A slice still not covered fails the order.
- The slices are combined, cut diagonally by default, and plated.
- This is the last point a bad spread can be fixed. Once the slices are together, a bare spot stays.

**Step 11: The machine checks and photographs the sandwich.** The final gate asks "Is this what the customer ordered?", then the photo goes on the order record and the order is ready.

- A failure here costs everything, and means an earlier gate is missing a check.

---

## Phase 3: Delivery

The handoff back to the customer. The sandwich crosses; the order stays with the machine as a record.

**Step 12: The machine tells the customer it's ready.** The progress screen and, when it can, a sound.

- The progress screen changes to say the sandwich is ready.
- A chime plays if audio is on.

**Step 13: The customer takes the sandwich.** The end of the path for the customer.

- The sandwich slides into the pickup area.
- The welcome screen comes back after 30 seconds with no one touching it.

**Step 14: The machine logs the order.** What it keeps, and what it leaves out on purpose.

- Kept: time to serve, the choices, which checks fired, and any problems fixed during the build.
- The data feeds operational dashboards.

Nothing about the person is kept. The machine and the sandwich are tracked, not who ordered.

---

## When an Order Fails

Every failure ends the order. The customer sees the specific reason, not a generic error, and is offered a new order, and the reason is logged with the step it happened at. What differs is where it stops and what has been used up by then.

| Fails at | The question it failed | What's used up |
|---|---|---|
| Step 7 | Do we have what we need, and is it good? | Nothing |
| Step 8 | Has the spread mixed? | Nothing but the stirring time |
| Step 10 | Did the spread and jelly go on properly? | The bread and the spread |
| Step 11 | Is this what the customer ordered? | Everything |
