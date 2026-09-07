# Feature Specs

A feature spec is an implementation-ready plan for one feature. It is layered — product altitude at the top, then behaviour, then data, then pseudocode — so a product reader can stop partway down and an engineer can keep going.

It describes something being built. That is what separates it from a core product flow, which describes something that already works.

## When to Use One

**Write one when a feature is agreed on and not yet designed in detail.**

- **The reader changes as the document goes down.** The top sections are read by everyone; the bottom by whoever implements it. Structure it so stopping early still leaves someone informed.
- **It is written before implementation, not after.** A spec written afterward is a flow with the wrong name.
- **One feature per spec.** A spec covering two features is one nobody can approve.
- **Not a roadmap.** Why this feature and not another belongs a layer up.

## Conventions

- **Open at product altitude.** What changes for the user, in language a non-technical reader can follow.
- **Descend in layers, and never jump back up.** Product, behaviour, data, implementation — a spec that returns to product framing after showing a schema has lost its reader.
- **State what is out of scope.** A spec that lists only what is included leaves every omission ambiguous.
- **Pseudocode over real code.** The point is the shape of the logic; real code goes stale the day it is written.
- **Name what already exists that this touches.** The registry, the pipeline, the endpoints — an engineer's first question is what they are changing rather than adding.

## Examples

| File | What It Covers |
|---|---|
| `fsp_artisan-spread-selection.md` | A user-facing feature layered from the selection experience down to registry changes and pipeline pseudocode |
