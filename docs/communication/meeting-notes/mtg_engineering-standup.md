# Engineering Standup — Day After the NutCo Outage

**Date:** 2026-03-19
**Attendees:** Dana Moss (engineering manager, facilitating), Riley Okafor (on-call), Jordan Kim (eng), Maya (eng), Tom (eng), Alex Rivera (eng), Casey Nguyen (product), one dial-in participant
**Type:** Standup — full engineering team
**Source:** Meeting-tool transcript with speaker labels. The dial-in participant is labeled only "Phone 1."
**Related:** `pm_peanut-butter-supplier-outage.md`, `pm_jelly-overapplication-degradation.md`, `tkt_order-service-registry-fallback.md`

# 📋 Summary

The daily engineering standup, held the morning after yesterday's NutCo supplier outage. Most of it covered outage follow-up: Riley is drafting the postmortem, and Dana kept supplier fallback out of ENG-315 so that it's handled in the postmortem instead. The other workstreams reported progress, and none are blocked apart from Jordan's quality alerting, which needs large-format bread orders in staging.

# 🗂️ Index

Each chapter is one workstream, in the order it was reported. The name on each chapter is the person who gave the update. Each outcome links to the chapter where it came up.

**Chapters**

1. [NutCo outage follow-up — Riley](#chapter-1-nutco-outage-follow-up--riley) — impact numbers and the postmortem draft
2. [Order service fallback — Alex](#chapter-2-order-service-fallback--alex) — ENG-315 progress, and whether it should cover suppliers
3. [Quality alerting — Jordan](#chapter-3-quality-alerting--jordan) — the INC-2301 alert, and a staging gap blocking it
4. [Spread adapter — Maya](#chapter-4-spread-adapter--maya) — interface draft on track; tempering still open
5. [Configuration storage — Tom](#chapter-5-configuration-storage--tom) — write-up due tomorrow
6. [Product — Casey](#chapter-6-product--casey) — checkout error copy and customer outreach

**Outcomes**

| Mark | Kind |
|---|---|
| ✅ | Decision |
| ❓ | Open question |
| 💡 | Idea |
| 👉 | Action item |

- ✅ ENG-315 stays scoped to the spread registry. Supplier API fallback becomes a postmortem action item instead. Decided by Dana. → [Chapter 2](#chapter-2-order-service-fallback--alex)
- ❓ During a supplier outage, should orders containing the affected spread be queued for later fulfillment or rejected with an explanation? Owner: Casey. Due: the postmortem review. → [Chapter 1](#chapter-1-nutco-outage-follow-up--riley)
- ❓ Can someone else take the staging expansion this week so the alert isn't held up? Owner: not stated — Jordan asked, and no one took it. → [Chapter 3](#chapter-3-quality-alerting--jordan)
- 👉 Riley — postmortem draft ready for review — by March 21. → [Chapter 1](#chapter-1-nutco-outage-follow-up--riley)
- 👉 Pull the support ticket count and top complaint categories from the outage window. Owner: not stated — the dial-in participant labeled "Phone 1." → [Chapter 1](#chapter-1-nutco-outage-follow-up--riley)
- 👉 Dana — schedule the postmortem review and invite Casey. → [Chapter 6](#chapter-6-product--casey)

# 💬 Conversation

The standup, split into one chapter per workstream, in the order each was reported. Each chapter opens with a summary of the update.

## Chapter 1: NutCo outage follow-up — Riley

Riley summarized yesterday's outage and is drafting the postmortem. The team's main question is what customers should experience the next time a supplier goes down.

- NutCo's inventory API was down for 2 hours 11 minutes yesterday, 10:22–12:33 UTC.
- 1,203 orders were rejected at checkout with a generic error. No payments were collected for them.
- The postmortem draft will be ready for review by Friday, March 21.
- Riley framed the core gap as having no fallback: one supplier API failing took out every order containing peanut butter.
- Queueing affected orders or rejecting them with a clear explanation were both raised. Dana asked Casey to own that choice, because it's a customer-experience decision.
- The dial-in participant offered to pull the support ticket count and top complaint categories from the outage window.

❓ **Open:** During a supplier outage, should orders containing the affected spread be queued for later fulfillment or rejected with an explanation? Owner: Casey. Due: the postmortem review.

## Chapter 2: Order service fallback — Alex

Alex reported progress on ENG-315 and asked whether it should also cover supplier outages. Dana kept the ticket's scope as it is.

- ENG-315, graceful degradation when the spread registry is unavailable, is in progress. Alex expects it in review next week.
- Alex pointed out that yesterday's outage is the same failure pattern one level down: a dependency fails, and the order service rejects orders instead of degrading.
- Alex asked whether ENG-315 should expand to cover supplier APIs.
- Dana said no. Expanding the scope would delay a fix that is already in progress, and supplier fallback belongs in the postmortem's action items.

✅ **Decision:** ENG-315 stays scoped to the spread registry. Supplier API fallback is handled as a postmortem action item. Decided by Dana.

## Chapter 3: Quality alerting — Jordan

Jordan is building the trend-based quality alert from the jelly overapplication postmortem. It works against historical data, but it can't be tested end to end until staging has large-format bread orders.

- The alert pages on-call if the 3-day average quality score drops more than 0.3 points below the prior 7-day average.
- Replayed against March data, it would have fired on March 5, two days before the weekly metrics review caught the jelly issue.
- End-to-end testing needs large-format bread orders in staging, and staging currently has none.
- Adding them is Jordan's other action item from the same postmortem, and it hasn't started. Jordan asked whether someone else could take it this week so the alert isn't held up. No one took it during the standup.

❓ **Open:** Can someone else take the staging expansion this week so the alert isn't held up? Owner: not stated — Jordan asked, and no one took it.

## Chapter 4: Spread adapter — Maya

Maya's spread adapter interface draft is on track for March 22. Where tempering logic lives is still open.

- The interface draft and peanut butter migration plan are on track for March 22.
- The tempering question, adapter layer or core controller, is still open. It was last discussed at the March 15 alignment.
- There was no new discussion of tempering during the standup.

## Chapter 5: Configuration storage — Tom

Tom's write-up on where spread configuration should live is due tomorrow, and Tom is leaning toward the database.

- The write-up is due March 20.
- Tom is leaning toward storing spread configuration in the database, so that adding a spread doesn't need a deploy.
- Tom will include the migration cost in the write-up.

## Chapter 6: Product — Casey

Casey reported on customer-facing follow-up from both recent incidents and asked to join the outage postmortem review.

- The checkout error during yesterday's outage was a generic "Something went wrong." Casey wants spread-specific copy regardless of how the queue-or-reject question is settled.
- Proactive outreach to the 847 customers affected by the jelly issue is still under discussion. Casey is not yet confident in the figure.
- Casey asked to be at the postmortem review, since the queue-or-reject question is Casey's to answer. Dana will schedule it and send Casey an invite.
