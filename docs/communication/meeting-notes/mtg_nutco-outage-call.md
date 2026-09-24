# NutCo Outage Follow-Up Call — Riley and NutCo

**Date:** 2026-03-20
**Attendees:** Riley Okafor (PB&J, on-call engineer), Marcus (NutCo account manager; first name only, as Riley used it)
**Type:** External call — supplier, incident follow-up
**Source:** Phone call recorded on Riley's end, with Marcus's consent. Only Riley's microphone was captured. Marcus's words are not in the source, so Marcus's positions appear here only as Riley restated or responded to them.
**Related:** `pm_peanut-butter-supplier-outage.md`, `mtg_engineering-standup.md`

# 📋 Summary

Riley called NutCo's account manager two days after the NutCo outage to gather NutCo's side for the postmortem draft due the next day. The call was tense over why NutCo's on-call line went unanswered during the outage. As Riley restated it, NutCo's position is that the line is only for contracted enterprise accounts; Riley disputed that, and it was not resolved. Marcus appears to have offered two things: adding PB&J's on-call alias to NutCo's incident notifications, and documentation for an availability export that PB&J could cache as a fallback. Riley did not commit to anything contractual, and deferred those questions to PB&J's manager.

# 🗂️ Index

Each chapter follows one thread of the conversation, listed in the order the thread first came up. Each outcome links to the chapter where it came up.

**Chapters**

1. [What happened on NutCo's side](#chapter-1-what-happened-on-nutcos-side) — NutCo's explanation of the outage, as Riley restated it
2. [Reaching NutCo during an outage](#chapter-2-reaching-nutco-during-an-outage) — the unanswered on-call line, the status page, and notifications
3. [An availability export for caching](#chapter-3-an-availability-export-for-caching) — what PB&J could fall back on when the API is down

**Outcomes**

| Mark | Kind |
|---|---|
| ✅ | Decision |
| ❓ | Open question |
| 💡 | Idea |
| 👉 | Action item |

- ✅ PB&J's on-call alias is added to NutCo's incident notifications. Offered by Marcus as Riley restated it; Riley accepted. Marcus's confirmation is not in the recording. → [Chapter 2](#chapter-2-reaching-nutco-during-an-outage)
- ❓ Is NutCo's on-call line available to PB&J during an outage? NutCo's position, as Riley restated it, is that it serves contracted enterprise accounts only. Riley disputed that. Owner: not stated — Riley deferred it to the contract renegotiation. → [Chapter 2](#chapter-2-reaching-nutco-during-an-outage)
- ❓ Is the hourly export worth moving to NutCo's higher tier? Owner: Riley, to bring to Riley's manager. → [Chapter 3](#chapter-3-an-availability-export-for-caching)
- 💡 Seed PB&J's supplier availability cache from NutCo's periodic export, so orders can fall back on recent stock data when the API is unreachable. → [Chapter 3](#chapter-3-an-availability-export-for-caching)
- 👉 Riley — email Marcus a written summary of the call today. → [Chapter 3](#chapter-3-an-availability-export-for-caching)
- 👉 Send the availability export documentation. Owner: Marcus, as Riley restated the offer. → [Chapter 3](#chapter-3-an-availability-export-for-caching)

# 💬 Conversation

The call, split into one chapter per thread, in the order each thread first came up. The notifications thread returned near the end, and that material is folded into Chapter 2. Each chapter opens with a summary of what it covered. Only Riley's side was recorded, so every attribution to Marcus is Riley's restatement.

## Chapter 1: What happened on NutCo's side

Riley asked what caused the outage, for the postmortem. As Riley restated it, NutCo attributed it to a database failover during planned maintenance that took far longer than expected.

- Riley opened by explaining the purpose: a postmortem draft due the next day, and wanting NutCo's account in it rather than a guess.
- As Riley restated it, NutCo's API went down when a database failover during planned maintenance stalled. Recovery took just over two hours.
- Riley asked whether the maintenance had been announced to API customers. Marcus appears to have said it was announced on NutCo's status page the week before. Riley said PB&J had not seen it and asked whether announcements go anywhere other than the status page.
- Riley repeated back NutCo's timeline to confirm it: the API went down at 10:22 UTC, and service was restored at 12:28.

## Chapter 2: Reaching NutCo during an outage

This was the tense part of the call. Riley pressed on why the on-call line went unanswered and why the status page stayed quiet for over twenty minutes. The dispute over who the on-call line serves was not settled. Near the end of the call, Marcus appears to have offered to add PB&J to NutCo's incident notifications, and Riley accepted.

- Riley said the on-call line was called at 10:31 UTC and nobody answered, so Riley filed a ticket through the vendor portal instead.
- As Riley restated it, NutCo's position is that the on-call line is for contracted enterprise accounts only, and PB&J is not one.
- Riley disputed that. The number is listed in the vendor portal with no such restriction, and nothing told PB&J the line wouldn't be answered.
- Riley said the status page posted nothing until 10:45, twenty-three minutes into the outage, and that PB&J spent that time working out whether the problem was on its own side.
- Marcus appears to have apologized for the status page delay and said it was being reviewed internally. Riley said PB&J would note NutCo's review in the postmortem.
- Riley said contract terms, including an SLA and an escalation contact, are being handled by PB&J's manager, and did not negotiate them on the call.
- Near the end of the call, Marcus appears to have offered to add a PB&J address to NutCo's incident notification list. Riley gave the on-call alias rather than a personal address, so the notifications reach whoever is on call.

✅ **Decision:** PB&J's on-call alias is added to NutCo's incident notifications. Offered by Marcus as Riley restated it; Riley accepted. Marcus's confirmation is not in the recording.

❓ **Open:** Is NutCo's on-call line available to PB&J during an outage? NutCo's position, as Riley restated it, is that it serves contracted enterprise accounts only. Riley disputed that. Owner: not stated — Riley deferred it to the contract renegotiation.

## Chapter 3: An availability export for caching

Riley asked what inventory data PB&J could keep locally, so that a NutCo outage no longer takes peanut butter off the menu. As Riley restated it, NutCo offers a daily availability export on PB&J's current tier and an hourly one on a higher tier.

- Riley explained that PB&J is building a short-lived local cache of supplier availability, so an unreachable supplier no longer means a rejected order.
- Riley asked whether NutCo offers anything bulk that PB&J could pull on a schedule, rather than calling the API on every order.
- As Riley restated it, NutCo offers a daily availability export as a CSV on PB&J's current tier, and an hourly export on a higher tier.
- Riley said a daily export is too stale to take orders against, but an hourly one might be usable.
- Riley asked about the price difference and said it would have to go to Riley's manager. Nothing was committed on the call.
- Marcus appears to have offered to send the export documentation. Riley said a written summary of the call would go to Marcus that day, so both sides have the same record.

❓ **Open:** Is the hourly export worth moving to NutCo's higher tier? Owner: Riley, to bring to Riley's manager.
