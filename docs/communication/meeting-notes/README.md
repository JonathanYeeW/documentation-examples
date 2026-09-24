# Meeting Notes

A meeting note is the written record of one meeting, made from its transcript. It replaces attendance. Someone who reads it, human or model, knows what was discussed, what was decided, and what was left open, as if they had been in the room, without rereading the transcript.

A note is usually written by an LLM from the transcript, with one of these examples as its template. That is why most of the conventions below are about what a note must not claim. A summary that reads well but states more than the recording supports is the failure a note is most prone to, and it is the hardest one for a later reader to detect.

## When to Write One

**One note per meeting.** Any meeting with a recording or transcript qualifies: a 1:1, a small group, a standup, a full-team discussion, or a personal conversation.

- **A status update is not a meeting note.** A status update is written for people who were not in any meeting and is organized by workstream state. A meeting note records a conversation that happened.
- **A postmortem is not a meeting note,** even when a meeting produced it. The postmortem owns the incident; the note records the meeting about it.
- **A decision's reasoning belongs to the document that owns the decision.** When a meeting settles something an exploration, plan, or TDD depends on, the note records the decision, and the owning document is updated separately. A note is not where a later reader should have to go to learn why a system is built the way it is.

## The Frame

The frame is the same for every meeting. What varies is which kinds of outcome appear.

```
Title
Metadata        Date · Attendees · Type · Source · Related (optional)

📋 Summary      Prose. What the meeting was, what it produced, what is left open.

🗂️ Index
  Chapters      Numbered links, one line describing each
  Outcomes      Key table, then one list: ✅ decisions · ❓ open questions · 💡 ideas · 👉 action items,
                each linking to its chapter

💬 Conversation One sentence on how the chapters are cut
  Chapter N     A summary paragraph for the whole chapter, then bullets.
                Any ✅ or ❓ from the chapter closes it as an inline line, with its reasoning above it.
```

Action items and ideas appear only in Outcomes. Decisions and open questions appear twice: as one line in Outcomes, and as a full statement where they happened. The Outcomes line is for finding them; the inline line keeps each one next to its reasoning.

## Conventions

### Fidelity

- **The note never states more than the transcript supports.** A tentative remark is not a decision. A date nobody said is not a due date. "Someone should look at that" is not an action item with a name on it.
- **"No decisions were made" is stated in the Summary** when it is true. That way the absence of ✅ reads as a fact, not an oversight.
- **Claims about the outside world are recorded as stated, not verified.** When a meeting is full of them, one line under Conversation says so.
- **A garbled term is flagged where it appears,** with the reading used. It is never silently resolved.
- **Fidelity is judged against the raw transcript, not a cleaned-up version.** Transcription tools that tidy the text can fill a gap with a plausible name or word the recording never contained. When a cleaned version and the raw one disagree, the raw one wins, and the difference is flagged.

### The Source Line

The Source line is how a reader calibrates trust in every attribution below it. It states what the recording can and cannot support:

- whether the transcript has speaker labels
- whether it begins mid-conversation
- whether only one side was recorded
- where the date came from, if not the transcript

### Attribution

- **A point is attributed to a person only when the transcript makes the speaker unambiguous.** That means a speaker label, a name addressed directly, or a role only one attendee holds. In a 1:1 this is nearly always possible. In a group without labels it rarely is, and the narrative stays unattributed.
- **In a one-sided recording, the unrecorded party speaks only through restatement.** Write "as Riley restated it" or "appears to have said," never a direct claim. A ✅ that only the recorded side stated says the other side's response is not in the source.
- **A decision made by one person with the authority to make it names that person:** "Decided by Dana."

### Owners

- **An owner is named only when the transcript makes it unambiguous.** Otherwise write `Owner: not stated —`, followed by whatever the transcript does identify, such as "the dial-in participant labeled Phone 1" or "Jordan asked, and no one took it."
- **A missing owner is better than a wrong one.** The description after the dash is what lets a reviewer fill it in.

### Summary

- **The Summary is one prose paragraph, even when the meeting covered several unrelated items.** It is read in full to decide whether to read the rest, so it isn't built for skimming. Anything that needs scanning belongs in the Index.

### Chapters

- **One chapter per thread, ordered by where the thread first came up.** When the conversation returns to a thread, the later material joins that chapter rather than opening a new one. Strict chronology would split a thread that circles back across three chapters.
- **A standup is chaptered by workstream.** The heading carries the name of the person who gave the update: `Chapter N: Workstream — Name`.
- **Every chapter opens with a paragraph summarizing the whole chapter,** then bullets. This deliberately departs from the review standard's two-sentence transition (C5). A reader skimming a long note reads the chapter summaries and stops.
- **One fact per bullet.** Options that were compared go in a table. Bold inline labels group sub-lists.

### Voice

- **Past tense, third person, neutral register.** A meeting note is a record, so the review standard's rule against past-tense session framing (B5) does not apply. This holds even when the writer was in the meeting and the conversation was personal.
- **Positions are recorded, not judged.** Profanity is paraphrased.

### Outcome Kinds by Meeting

The key table always lists all four kinds, even when a meeting produced only some of them.

- A technical alignment produces ✅ and ❓.
- A brainstorm produces 💡, and usually no ✅.
- A standup produces 👉, with blockers as ❓.

## Examples

| File | What It Covers |
|---|---|
| `mtg_spread-integration-architecture.md` | Technical alignment, small group. Options compared in a table, and two decisions with their reasoning inline. Owners are named because work was assigned by name. The narrative is unattributed because the transcript has no speaker labels |
| `mtg_engineering-standup.md` | Full-team standup, chaptered by workstream. Speaker labels make attribution safe, and a decision is credited to the person who made it. Shows the owner fallback twice: an unlabeled dial-in participant, and a request nobody took |
| `mtg_quality-ratings-brainstorm.md` | 1:1 brainstorm with no agenda and no decisions, where ideas carry the outcomes. The recording begins mid-conversation and attribution is partial. A thread that returned near the end is folded into its chapter |
| `mtg_nutco-outage-call.md` | External call recorded on one end, with tension. The other party speaks only through restatement, and a ✅ carries a qualifier because only one side is on the record. A disagreement is recorded neutrally and left open, and a thread that returned near the end is folded into its chapter |
