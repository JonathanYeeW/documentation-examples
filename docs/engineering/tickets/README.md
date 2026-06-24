# Tickets

A ticket is a scoped unit of work — what needs to be built or fixed, and why. Each ticket tells the problem story: what's wrong or missing, how a person experiences it today, and what the world looks like after it's resolved.

## Format

Ticket titles follow the conventional commit prefix pattern (`feat:`, `bug:`, `refactor:`, `build:`, etc.), all lowercase.

Each ticket has three sections:

**🔴 The Problem** — one or two sentences. The gap between now and where we want to be, in plain language.

**📍 Experience Today** — a short flow showing how a person encounters the problem right now.

**✅ Experience After** — a short flow showing what the experience looks like once the ticket is resolved.

The two experience sections use code blocks to represent the flow, consistent with CPF documents.

## Examples

| File | What It Covers |
|---|---|
| `tkt_disable-spread-availability.md` | Feature — admin needs to temporarily disable a spread without deleting it from the registry |
| `tkt_order-service-registry-fallback.md` | Reliability — order service fails hard when spread registry is unavailable instead of falling back to cache |
| `tkt_spread-selector-state-fix.md` | Bug — spread selector loses selection when user navigates away and returns |
