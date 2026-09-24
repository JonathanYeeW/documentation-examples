# Spread Integration Architecture — Technical Alignment

**Date:** 2026-03-15
**Attendees:** Maya (eng), Tom (eng), Priya (product)
**Type:** Technical alignment — small group
**Source:** Meeting transcript, no speaker labels. Owners are named where the transcript assigns work by name.
**Related:** `cpf_assembly-pipeline.md`, `fsp_artisan-spread-selection.md`

# 📋 Summary

The team met to decide how the assembly controller should support spread types beyond peanut butter. They decided to build a generic spread adapter layer and migrate peanut butter onto it first, so the pattern is proven on known-good behavior before any new spread ships. Two questions are open: where temperature pre-handling logic lives, and whether spread configuration is stored in the database or in application code.

# 🗂️ Index

Each chapter follows one thread of the conversation, listed in the order the thread first came up. Each outcome links to the chapter where it came up.

**Chapters**

1. [Why the controller has to change](#chapter-1-why-the-controller-has-to-change) — the Q2 spread goal, and peanut butter logic tangled through the controller
2. [Choosing an approach](#chapter-2-choosing-an-approach) — three options weighed; the adapter layer chosen
3. [What the adapter needs to know](#chapter-3-what-the-adapter-needs-to-know) — viscosity, temperature, a setup step; where tempering lives is open
4. [Where spread configuration lives](#chapter-4-where-spread-configuration-lives) — database or application code; open
5. [Sequencing](#chapter-5-sequencing) — peanut butter first, then almond butter

**Outcomes**

| Mark | Kind |
|---|---|
| ✅ | Decision |
| ❓ | Open question |
| 💡 | Idea |
| 👉 | Action item |

- ✅ Build a generic spread adapter layer, with peanut butter migrated onto it first. → [Chapter 2](#chapter-2-choosing-an-approach)
- ✅ Almond butter is the first new spread on the adapter, after the peanut butter migration is reviewed and tested. → [Chapter 5](#chapter-5-sequencing)
- ❓ Does temperature pre-handling live in the adapter layer or the core controller? Owner: Maya. Due: before almond butter implementation begins. → [Chapter 3](#chapter-3-what-the-adapter-needs-to-know)
- ❓ Is spread configuration stored in the database or in application code? Owner: Tom. Due: before this part of the system is built, not before the adapter work begins. → [Chapter 4](#chapter-4-where-spread-configuration-lives)
- 👉 Maya — draft the spread adapter interface and the peanut butter migration plan — by March 22. → [Chapter 5](#chapter-5-sequencing)
- 👉 Tom — write up the configuration storage options with a recommendation — by March 20. → [Chapter 4](#chapter-4-where-spread-configuration-lives)
- 👉 Priya — confirm the Q2 spread priority order with the operations team, so engineering knows which spread follows almond butter — by March 18. → [Chapter 5](#chapter-5-sequencing)

# 💬 Conversation

The conversation, split into one chapter per thread, in the order each thread first came up. Each chapter opens with a summary of what it covered.

## Chapter 1: Why the controller has to change

More spreads are coming in Q2 and Q3, and peanut butter support is tangled through the assembly controller. Adding spreads the same way would make the controller unmaintainable, so the architecture has to be decided now.

- The Q2 product goal is that customers can select spread types beyond peanut butter when building a sandwich.
- Almond butter and sunflower butter are both queued for Q2. Four more spreads are on the Q3 roadmap.
- Peanut butter support is tangled throughout the assembly controller, the part of the system that manages the sandwich-making sequence.

## Chapter 2: Choosing an approach

Three architectures were weighed for adding spreads. The team chose a generic adapter layer and accepted the upfront cost of migrating peanut butter onto it first.

| Option | How it works | Outcome |
|---|---|---|
| Generic spread adapter layer | A layer between the controller and the spreads. Every spread plugs in the same way, and the controller talks only to the layer. | Chosen |
| Per-spread conditional logic | Keep adding logic to the controller for each new spread. Faster in the short term. | Ruled out early |
| Hybrid | Leave peanut butter's logic in place, build the adapter for new spreads only, and migrate peanut butter later. | Rejected |

- Per-spread logic was ruled out because the controller is already hard to follow with one special-cased spread, and every spread added makes it worse.
- The hybrid avoids touching working peanut butter logic, but it has two problems:
    - Peanut butter becomes a permanent exception every engineer has to know about.
    - The adapter's first real test would be a new, unproven spread rather than one with a production track record.
- The adapter layer costs more upfront. In exchange, a flaw in the adapter surfaces during the peanut butter migration rather than during almond butter's launch week.

✅ **Decision:** Build the generic spread adapter layer. Peanut butter is migrated onto it first, not a new spread, so the pattern is proven on known-good behavior before almond butter ships.

## Chapter 3: What the adapter needs to know

The team listed what the adapter needs from each spread. Temperature raised a design question that stayed open: whether tempering belongs in the adapter or in the controller.

- The adapter needs three things from each spread:
    - **Viscosity** — how much pressure the spreader applies.
    - **Temperature requirements** — whether the spread needs preparation before it can be applied.
    - **A setup step** — run by the controller before the assembly sequence begins, to handle spread-specific preparation.
- Some spreads have to be tempered to room temperature before use. There are two places that logic could live:
    - **In the adapter layer** — keeps the controller clean, but the adapter has to understand the machine's heating infrastructure.
    - **In the core controller** — keeps machine-level concerns in one place, but couples the controller to spread-specific behavior.

❓ **Open:** Does temperature pre-handling live in the adapter layer or the core controller? The answer sets how much the adapter needs to know about the machine's physical infrastructure. Owner: Maya. Due: before almond butter implementation begins.

## Chapter 4: Where spread configuration lives

The controller needs each spread's properties while a sandwich is being made. The team compared storing them in the database with storing them in code, and left the choice to Tom's write-up.

- The controller looks up spread properties such as viscosity and temperature requirements during assembly.
- **In the database** — adding a spread needs no code change and can happen at any time.
- **In application code** — simpler to set up, but every new spread needs a code change and a deployment.

❓ **Open:** Is spread configuration stored in the database or in application code? Owner: Tom. Due: before this part of the system is built, not before the adapter work begins.

## Chapter 5: Sequencing

The team agreed the order of work and assigned the next steps. Two real spreads have to run on the adapter before the pattern is considered stable.

- Build the adapter layer and migrate peanut butter, then get it reviewed and tested.
- Implement almond butter as the first new spread on the adapter.
- Engineering needs the Q2 priority order to know which spread follows almond butter.

✅ **Decision:** Almond butter is the first new spread on the adapter, after the peanut butter migration is reviewed and tested.
