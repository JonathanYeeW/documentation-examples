# Machine Control

**Created:** 2026-02-03
**Updated:** 2026-08-19

## Summary

This directory holds everything about driving a PB&J machine — the command protocol it speaks, the sensors it reports from, and the safety interlocks that can refuse a command. It does not cover what a sandwich is or how an order is priced; those are product concerns and live one level up.

Read it before writing anything that talks to a machine. Each child covers one half of a conversation with a physical device, and the things that make that conversation work are properties of the device rather than of any one protocol.

## What's Here

| Directory | What It Covers |
|---|---|
| [`protocol/`](protocol/) | The command protocol — framing, the command set, acknowledgements, and how a refusal is reported |
| [`sensors/`](sensors/) | Reading machine state — the sensor set, polling intervals, and what an absent reading means |
| [`interlocks/`](interlocks/) | The safety interlocks, what each one guards, and the order they are evaluated in |
| `firmware/` *(planned)* | Firmware versions, what a version change can break, and how a fleet is upgraded |

## Good to Know

**Every machine on the fleet runs the same firmware version, and that is enforced rather than assumed.** A machine that falls behind is taken out of rotation rather than driven by a compatibility branch, because a protocol that has to work against two firmware versions is a protocol nobody can reason about. This is the constraint the whole directory rests on and it belongs to none of the children.

**Machine identifiers are site-scoped, not global.** Machine 3 at one site and machine 3 at another are different devices. Every document here uses the full `<site>/<machine>` form, and code carrying a bare machine number is code that will eventually address the wrong device.

**A machine accepts one control session at a time.** Two processes talking to one machine is not a race to be handled but a configuration error to be prevented, and no child document defends against it.
