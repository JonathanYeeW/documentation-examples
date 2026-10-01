# The Order Lifecycle

**Created:** 2026-04-11

# 📋 Summary

Every order moves through a fixed set of states, from the moment a customer taps **Make My Sandwich** to the moment the sandwich is collected or the order fails. This article covers each state, how an order moves between them, and what the customer sees. How the machine builds the sandwich, and what the customer did before ordering, are covered elsewhere.

# 🧭 Context

**What the PB&J Machine is for**

Kids go hungry because the systems that feed them are slow, expensive and fragile. PB&J Machine Co. builds the software for a machine that can make any peanut butter and jelly sandwich on demand: a parent, a cafeteria worker or a food bank volunteer picks a bread, a spread and a jelly, and the machine makes it. The goal is a sandwich for any kid who needs one, with no training and no waste.

**Where this sits in the journey**

- A customer builds a sandwich in the order flow, then taps **Make My Sandwich**.
- The machine builds it, puts it in the pickup area, and tells the customer it's ready.
- Throughout, the customer watches a **progress screen** showing where their order is.

**Terms**

- **Make My Sandwich** — the button at the end of the order flow that sends the order to the machine.
- **Progress screen** — the screen a customer watches after ordering, until the sandwich is ready.
- **Pickup area** — the tray where a finished sandwich waits to be collected.

**Not covered here**

- How the machine builds a sandwich, step by step. That's in the assembly log and the quality model.
- What the customer did before ordering, like which screens they visited. That's session state.

# ⚙️ Section 1: How an Order Moves

An order is always in exactly one **state**, and moves to the next when something happens on the machine. This section is the model every state in Sections 2 and 3 shares.

**How a state changes**

- **An order moves on an event, not a timer.** It moves when the machine reports that something happened, so its state is always where it actually is.
- **Ready to collected is the one timed move.** After 5 minutes the order is collected, because the pickup sensor can't tell a collected sandwich from an abandoned one, and reporting doesn't need to.
- **Only problems the machine can't fix reach the order.** A **recoverable failure**, like a coverage gap the machine touches up, is fixed during the build, and the order never sees it.
- **Two states are terminal:** collected and failed. An order never leaves them.
- **An order tracks the machine and the sandwich, not the person.**

**An example: one sandwich**

```
Tap Make My Sandwich               → submitted
Bread leaves storage               → assembling    (80 seconds, one touch-up)
Plated and photographed            → ready         (chime plays)
Picked up after 2 minutes          → collected     (time to serve logged)
```

An order ends one of two ways, and Sections 2 and 3 follow them: on its way to the customer, or failed.

# 🥪 Section 2: On Its Way to the Customer

The first way an order ends: everything works, and the customer gets their sandwich.

## 2.1 Submitted

The customer has confirmed their sandwich, and nothing physical has happened yet.

- **The customer sees:** the progress screen.
- **Moves on when:** the machine picks the order up, usually straight away.

## 2.2 Assembling

The sandwich is being built, from ingredients leaving storage to the finished sandwich being plated and photographed.

- **The customer sees:** the progress screen.
- **Moves on when:** the sandwich is plated and its quality photo is taken, usually after 60–90 seconds.
- Quality checks run as early as they can, so a problem wastes as little as possible.

## 2.3 Ready

The sandwich is in the pickup area.

- **The customer sees:** the progress screen saying it's ready, and hears a chime if audio is on.
- **Moves on when:** the sandwich is picked up, or 5 minutes pass.

## 2.4 Collected

The order finished. It's terminal.

- **The customer sees:** nothing more. The order is done.
- **Logged:** time to serve, the selections made, which quality checks triggered, and any problems fixed during the build.

# ⚠️ Section 3: Failed

The second way an order ends: the machine hits a problem during assembling that it can't fix.

## 3.1 Failed

The order stopped before a sandwich was made. It's terminal.

- **The customer sees:** the specific reason, like "We couldn't get an even spread on your sandwich", not a generic error, and an offer to start a new order.
- **Logged:** the reason, the step it happened at, and any ingredients wasted.
