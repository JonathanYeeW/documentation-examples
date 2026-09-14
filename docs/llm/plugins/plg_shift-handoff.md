# Plugin — Shift Handoff

**Created:** 2026-03-14

# 📋 Summary

This plugin writes a handoff entry to the shift log at the end of a session that changed a machine's configuration. It runs during offboarding. The reader of what it produces is the next operator on shift, who was not present for any of the work and needs to know what changed before they take the first order.

# 🔔 Trigger

A machine's configuration changed during the session.

- A calibration pass completed.
- A spread, bread, or jelly config was added, edited, or removed.
- A machine was taken out of the queue or returned to it.

If none of these happened, skip the plugin. A session that only read state produces no handoff entry.

# 🧾 Phases

Two phases. Phase 1 gathers what changed, phase 2 writes it. The gate between them exists because the operator is the only one who knows which changes were intentional.

## Phase 1 — Collect the Changes

Assemble the list of configuration changes from the session, in the order they happened. For each one, capture:

- The machine identifier.
- What changed, stated as the new state rather than as the edit.
- Whether the machine is currently in the order queue.

Present the list and ask whether anything is missing or was undone later in the session.

> ⏸️ **GATE → Operator.** Confirm the list before it is written. A handoff entry that names a change the operator reverted is worse than no entry.

## Phase 2 — Write the Entry

Append the entry to `shift-log/YYYY-MM-DD.md`. Create the file if the day has no log yet.

- One entry per session, not one per change.
- Machines are listed in identifier order, not in the order they were touched.
- A machine out of the queue is named first, whatever its identifier. That is the fact the next operator acts on.
- The entry is appended. An existing entry from an earlier session that day is never edited.

# 💬 Example

**Operator:** offboard

**Claude**

Three configuration changes this session. Confirm before I write the handoff.

- `line-2` — spread pressure recalibrated for the new almond butter supplier. In the queue.
- `line-4` — taken out of the queue. The cut stage overshoots by 3mm and the fix is not in yet.
- `bench-a` — added `fig-jam` to the jelly registry for next week's trial. In the queue.

Anything missing, or anything here that got undone later?

**Operator:** all correct

**Claude**

Appended to `shift-log/2026-03-14.md`.

```
## Session — 14:20 to 17:45

- line-4 — OUT OF QUEUE. Cut stage overshoots by 3mm. Fix not deployed.
- line-2 — Spread pressure recalibrated for the new almond butter supplier. In queue.
- bench-a — fig-jam added to the jelly registry for next week's trial. In queue.
```

`line-4` is first because it is the one that changes what the next operator can do.
