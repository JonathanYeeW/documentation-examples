# Quality Ratings Brainstorm — Casey and Jordan

**Date:** 2026-03-12
**Attendees:** Casey Nguyen (product), Jordan Kim (eng)
**Type:** 1:1 — brainstorm, no agenda
**Source:** Voice memo recorded on a phone. No speaker labels, and the recording begins a few minutes into the conversation. Points are attributed only where the speaker is clear from context.
**Related:** `pm_jelly-overapplication-degradation.md`, `kb-con_quality-model.md`

# 📋 Summary

An unstructured coffee conversation three days after the jelly overapplication fix shipped, about what quality ratings could do beyond catching incidents late. It produced four ideas: segmenting the weekly review by default, per-bread calibration profiles, one-tap rating reasons, and a credit instead of an apology email for affected customers. No decisions were made. Outreach to affected customers is still open, and two follow-ups came out of it.

# 🗂️ Index

Each chapter follows one thread of the conversation, listed in the order the thread first came up. Each outcome links to the chapter where it came up.

**Chapters**

1. [What the ratings caught](#chapter-1-what-the-ratings-caught) — why the jelly issue surfaced in ratings and not in alerts
2. [Segmenting the ratings](#chapter-2-segmenting-the-ratings) — which cuts of the data would have shown it sooner
3. [Per-bread calibration profiles](#chapter-3-per-bread-calibration-profiles) — making bread shape something every applicator reads
4. [Asking customers why](#chapter-4-asking-customers-why) — one-tap reasons for a low rating
5. [Reaching affected customers](#chapter-5-reaching-affected-customers) — outreach to the 847, and what form it takes

**Outcomes**

| Mark | Kind |
|---|---|
| ✅ | Decision |
| ❓ | Open question |
| 💡 | Idea |
| 👉 | Action item |

- 💡 Segment the weekly metrics review by bread type, spread, and machine by default, rather than only after something looks off. → [Chapter 2](#chapter-2-segmenting-the-ratings)
- 💡 A calibration profile per bread type that every applicator reads, with a new bread type blocked from launch until it has one. → [Chapter 3](#chapter-3-per-bread-calibration-profiles)
- 💡 Optional one-tap reasons ("soggy," "dry," "uneven," "wrong spread") shown only when a customer rates a sandwich 3 or lower. → [Chapter 4](#chapter-4-asking-customers-why)
- 💡 A credit on the next order for affected customers, instead of an apology email. → [Chapter 5](#chapter-5-reaching-affected-customers)
- ❓ Is the 847 figure reliable enough to contact customers at all? Owner: Casey. → [Chapter 5](#chapter-5-reaching-affected-customers)
- 👉 Casey — bring a segmented view to the next weekly metrics review, as a trial. → [Chapter 2](#chapter-2-segmenting-the-ratings)
- 👉 Pull the most common words from low-rating support tickets, as a starting list of reasons. Owner: not stated — offered by one attendee, and the recording doesn't make clear which. → [Chapter 4](#chapter-4-asking-customers-why)

# 💬 Conversation

The conversation, split into one chapter per thread, in the order each thread first came up. It returned to segmentation near the end, and that material is folded into Chapter 2. Each chapter opens with a summary of what it covered.

## Chapter 1: What the ratings caught

The jelly issue never tripped an alert. It surfaced as a slow drift in quality ratings, found in a weekly review four days after the deploy. The conversation treated ratings as the only signal that measures what a customer actually eats.

- The applicator update shipped on March 3. The weekly metrics review flagged the drop on March 7.
- The average quality score fell from 4.6 to 3.9 over the incident window. No single day dropped sharply enough to trip existing thresholds.
- Jordan described the fix: jelly volume is now calculated from bread surface area, not from a fixed pressure value.
- The quality gates check coverage and spec, not texture. A soggy sandwich passes every gate, so a rating is the only place sogginess shows up.

## Chapter 2: Segmenting the ratings

Segmenting by bread type found the jelly issue within hours, once someone thought to do it. The idea is to segment by default. The conversation returned to which segments matter near the end.

- The overall average hid the problem. Large-format bread orders were down 1.2 points while standard bread showed no change.
- The idea is that the weekly review shows ratings by segment by default, so a drop in one segment is visible without anyone going looking for it.
- Near the end, three candidate segments came up: bread type, spread, and machine.
- Machine was the most debated. It would catch a single miscalibrated unit, but the sample per machine per week may be too small to mean anything.
- Casey will try a segmented view at the next weekly review before anything is built.

## Chapter 3: Per-bread calibration profiles

The fix made bread surface area explicit for the jelly applicator. The idea is to generalize it into a profile per bread type that every applicator reads.

- The postmortem already has an action item to make bread surface area a first-class field in the order system.
- The idea extends that to a calibration profile per bread type: surface area, plus anything else an applicator needs to know.
- The spread applicator would read the same profile, so a new applicator couldn't quietly assume standard bread.
- The concern raised was that a profile only helps if it exists. The suggestion was that a new bread type can't launch without one.

## Chapter 4: Asking customers why

A rating says a sandwich was bad, but not why. The idea is a short set of optional reasons, shown only on low ratings, so customers who are happy aren't asked anything more.

- Customer feedback during the incident used the same few words repeatedly, such as "soggy" and "falling apart," but only in free-text support tickets.
- The idea is one-tap reasons, such as "soggy," "dry," "uneven," or "wrong spread," shown only when a rating is 3 or lower.
- Survey fatigue was the objection. Limiting reasons to low ratings was the answer given.
- One attendee offered to pull the most common words from low-rating support tickets as a starting list. The recording doesn't make clear which attendee.

## Chapter 5: Reaching affected customers

Casey owns the postmortem item on proactive outreach. The conversation didn't settle whether to reach out, but produced an idea for what form outreach could take.

- The open question is whether the 847 figure is reliable enough to contact customers. Contacting someone who wasn't affected would be confusing, and possibly alarming.
- The idea is a credit on the next order rather than an apology email. A credit reads as goodwill even to someone who didn't notice a problem.
- No decision was made. The outreach item stays with Casey.

❓ **Open:** Is the 847 figure reliable enough to contact customers at all? Owner: Casey.
