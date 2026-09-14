# Plugins

A plugin extends a session at a fixed point in it — typically onboarding or offboarding. It is either a standing constraint the model holds for the whole session, or a conditional procedure that runs only when its trigger is met.

A plugin exists because some rules cannot be applied retroactively. A convention about how every response is formatted, or a check that must happen before work starts, has to be loaded before the session does anything, not recalled halfway through.

## When to use one

**Write a plugin when something must be in place from the start of a session rather than invoked during it.**

- **A standing constraint** holds for the session's whole length — a format every response follows, a rule about which surfaces may be written to, a vocabulary to use.
- **A conditional procedure** loads with the session but only runs when its trigger fires. It is loaded early because by the time the trigger fires it is too late to go looking.
- **Not an SOP.** An SOP is invoked deliberately to produce an artifact. A plugin is loaded whether or not it ends up doing anything. See [`../sops/`](../sops/).
- **Not a project README.** A document that orients a model to a codebase or project belongs in [`../../readmes/projects/`](../../readmes/projects/).

## Conventions

### Structure

Three sections. A conditional plugin adds a fourth.

| Section | Required | What It Holds |
|---|---|---|
| `# 📋 Summary` | Always | Prose. What the plugin does, when it loads or runs, who reads it, and what they need from it |
| `# Trigger` | Conditional only | The condition that causes it to run, **and what happens when the condition is not met** |
| Body | Always | Phases for a procedure, grouped rules for a standing constraint |
| `# 💬 Example` | Always | Inline. A plugin is self-contained and does not depend on a document outside itself |

**Every heading carries a transition before its content** — what the section is and why it matters, in no more than two sentences.

**A trigger states both branches.** What runs when the condition is met, and what the session does when it is not. A trigger with only the positive branch leaves a model guessing whether to run anyway.

**The example is inline.** A plugin is loaded into a session in full, so an example that lives in another file is an example the model does not have.

### What does not go in

**Derivation does not belong in a plugin.** Why it was built this way lives in the ticket workspace that produced it. A plugin is loaded into every session that uses it, and reasoning the model does not need is context spent for nothing.

**Neither does anything that can be looked up when needed.** A plugin occupies the session from the moment it loads. If it can be fetched at the point of use, it should be.

## Examples

| File | What It Covers |
|---|---|
| [`plg_machine-targeting.md`](plg_machine-targeting.md) | The standing-constraint shape — grouped rules held for the session's whole length, no trigger |
| [`plg_shift-handoff.md`](plg_shift-handoff.md) | The conditional shape — a `Trigger` section stating both branches, and phases that run only when it fires. Copy this one when the plugin has a trigger |
