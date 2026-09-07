# Spread Standards

**Created:** 2026-04-14
**Updated:** 2026-07-02

## Summary

This directory holds the standards for anything that handles a spread — peanut butter, jelly, and the seasonal varieties. It covers how a spread is represented in the registry, how it is tempered before application, how it is measured, and what a machine does when it runs out mid-order.

Read it before writing code in a spread pipeline. The four standards below share a vocabulary and a unit, so a session that opens one in isolation will write code that is locally correct and wrong at the seams.

## What's Here

| File | What It Covers |
|---|---|
| [`registry.md`](registry.md) | The spread registry — required fields, the viscosity curve, and why a supplier change creates a new entry rather than editing one |
| [`tempering.md`](tempering.md) | Bringing a refrigerated charge to application temperature, the timeout, and what a pipeline does when it is not reached |
| [`measurement.md`](measurement.md) | Grams as the stored unit, the rounding rule, and how an interface converts for display |
| `depletion.md` *(planned)* | What happens when a charge empties mid-order — partial application, operator alert, and resume |

## Context

- **Stored unit:** grams, everywhere, including amounts an interface renders as spoons
- **Registry table:** `spreads`, one row per spread, shared across the whole fleet
- **Viscosity reference temperature:** 21°C — every curve in the registry is measured at it
- **Charge bays per machine:** 2
- **A spread** is a registry entry: an identifier, a viscosity curve, and a supplier
- **A charge** is one loaded cartridge of one spread
- **An application** is one dispense onto one slice

## Good to Know

**A viscosity value in the registry is measured at application temperature.** Using it against a cold charge produces a pressure wrong by roughly a factor of three. This is the most common defect in spread code and it belongs to no one document — the registry owns the value, tempering owns the temperature, and the mistake happens between them.
