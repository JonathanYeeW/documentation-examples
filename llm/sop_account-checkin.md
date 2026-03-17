# Account Check-In Prep Procedure

**Created:** 2026-03-07
**Purpose:** Produce a structured prep doc before a recurring account check-in call. Combines context from the account's CRM history, recent correspondence, and open tickets into a three-section briefing: Where We Are, Talking Points, and Gaps to Fill.

---

## Trigger

Account manager says: `"check-in prep for [account name]"`

Claude looks up the account in the CRM and begins Phase 1.

---

## Phase 1 — Load Context

Fetch the following from the CRM for the account:

1. **Account summary** — the most recent rolling summary for the account (use the latest approved entry)
2. **Most recent archived summary** — the last locked/archived summary for historical context
3. **Last 2 correspondence entries** — the most recent inbound and outbound interactions. Direction (inbound from account / outbound from us) will be self-evident from the content.

Once fetched, Claude synthesizes the material and presents a brief "here's where things stand" — 3–5 sentences covering:
- Where the account is in their relationship with PB&J Co. right now
- Where things stood at the end of the last interaction
- Anything that looks stale, unresolved, or worth flagging before the call

> ⏸️ **GATE → Account Manager.** React to the synthesis. Correct anything stale, add anything the CRM missed, or just say "looks good." Once confirmed, proceed to Phase 2.

---

## Phase 2 — Build the Doc

Using the confirmed context, Claude produces the check-in prep doc with three sections:

### Section 1: Where We Are
2–3 sentences of prose. Not bullets. Reads like briefing a colleague on the account — present tense, relationship-level altitude. Should take 30 seconds to read and leave the account manager fully present in the account relationship before the call starts.

### Section 2: Talking Points
Two subsections, minimal formatting. Each entry is a short anchor phrase — just enough to launch the conversation, not a full sentence to read aloud.

```
**Their world**
- [open issue / request / thing they raised last call]
- [follow-up on something from their last interaction]
- ...

**Our world**
- [update to share — in delivery order]
- ...
```

Their world comes first. The account manager is responding to the account's needs before sharing company news.

### Section 3: Gaps to Fill
3–5 short items — open questions or missing context that this call is an opportunity to resolve. Pulled from the CRM's open questions and any unresolved threads from the last two interactions. Framed as things to ask or listen for during the call.

---

## Phase 3 — Save the Doc

Save the finished doc to:

```
accounts/[account-slug]/YYYY-MM-DD-checkin-prep.md
```

Confirm the file path to the account manager when saved.

---

## Example

The following is a finished check-in prep doc produced by this procedure, using Grover's Market as the example account.

---

# Grover's Market — March 7, 2026

## Where We Are

Grover's just wrapped a six-week pilot of artisan spread selection across three stores and is waiting on final sales lift numbers before deciding whether to roll it out chainwide. The relationship is warm but the decision is sensitive — their category buyer Tom has been skeptical of the UI complexity from the start, and their ops lead Sandra had two unexplained machine restarts during the pilot. The last call ended with a commitment to deliver a root cause summary on those restarts before this one.

---

## Talking Points

**Their world**
- Root cause on Sandra's restarts — deliver the summary, don't wait for them to ask
- Pilot sales lift — do they have numbers yet, or are we still waiting together?
- Tom's UI skepticism — has his position shifted after seeing it in stores?

**Our world**
- Chainwide rollout timeline — if numbers look good, all 14 locations within two sprints
- Q2 spread catalog expansion — 12 new regional varieties, Grover's gets early access as a pilot account
- New account dashboard — Sandra gets direct visibility into machine uptime and restart logs

---

## Gaps to Fill

- What's driving Tom's skepticism — is it the UI specifically, or something deeper about pilot ROI?
- Does Sandra know about the new dashboard yet, or is this the first she's hearing it?
- Is there an internal deadline at Grover's for the chainwide decision, or is this open-ended?
