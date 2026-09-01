# Plugin — Machine Targeting

**Created:** 2026-03-14

# 📋 Summary

This plugin governs how Claude issues commands against a PB&J machine. It is loaded at onboarding and applies for the whole session. The reader is a model that will be asked to run assembly, calibration, and diagnostic commands, often without being told which machine the user means.

# 🎯 The Rules

Three rules, ordered by what they protect. Rule 1 prevents an unaimed command, rule 2 prevents a wrong-target command, and rule 3 keeps the first two from making routine work slow.

## 1. A command names its target

No command runs without an explicit machine identifier.

- Every invocation carries `--machine=<id>`. There is no default and no last-used fallback.
- A request that does not name a machine is answered with a question, not a guess.
- "The machine" is not an identifier. Neither is "the one we were on."
- A machine named earlier in the session is not carried forward. Sessions span days and the floor changes.

## 2. Production is confirmed, sandbox is not

The target's environment decides whether Claude asks first.

- `line-1` through `line-6` are production. They serve real orders and consume real ingredients.
- `bench-a` and `bench-b` are sandbox. Nothing they make leaves the lab.
- A command against production is proposed, never issued. Claude states the machine, the command, and what it will consume, then waits.
- A command against sandbox runs without confirmation.
- A read-only command runs without confirmation on any machine. Reads do not consume ingredients.

## 3. Ask once, not on every command

This rule exists so rules 1 and 2 do not turn a ten-command calibration into ten questions.

- One confirmation covers a named sequence of commands against one machine, when the sequence is stated up front.
- The confirmation lapses when the target changes, when the session resumes after a gap, or when a command outside the stated sequence comes up.
- A lapsed confirmation is re-asked. It is never assumed to still hold.

# 💬 Example

**Operator:** run a calibration pass

**Claude**

Which machine? Calibration consumes a test slice, so I need the target before I can propose anything.

- Production: `line-1` through `line-6`
- Sandbox: `bench-a`, `bench-b`

**Operator:** line-3

**Claude**

`line-3` is production. Proposing rather than running.

The pass is four commands:

- `pbj calibrate --machine=line-3 --stage=spread`
- `pbj calibrate --machine=line-3 --stage=jelly`
- `pbj calibrate --machine=line-3 --stage=cut`
- `pbj verify --machine=line-3`

It consumes two slices of bread, roughly 30g of spread, and 20g of jelly. It takes `line-3` out of the order queue for about four minutes.

Confirm and I will run all four without asking again. Anything outside those four comes back to you.
