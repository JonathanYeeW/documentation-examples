# PB&J Machine Co. — Quarterly Plan

**Created:** 2026-07-01
**Quarter:** Q3 2026 (July 1 – September 30)
**Product Vision:** `pv_pbj-co.md`

# 📋 Summary

Q3 is the quarter PB&J Machine Co. stops requiring a sales call to activate a new customer. The four requirements — machine connection, batch scheduling, dietary configuration, and self-serve onboarding — combine into a complete self-serve experience that takes a school from zero to producing sandwiches in a single session. Two stretch goals (parent visibility and usage reporting) ship if requirements land early. The work is sequenced by dependency: each phase builds on the one before it, with onboarding last because it wraps everything else.

# 🎯 Goal

By end of Q3, any school cafeteria in the US can sign up, connect their machine, and start producing sandwiches without help from our team. Self-serve onboarding is live. Batch scheduling handles a full week of orders. At least three pilot schools are running production sandwiches through it.

## How This Fits the Product Vision

The long-term vision is a PB&J machine in every place a kid might be hungry. The strategic bet is distribution — getting machines into schools, food banks, and community centers at scale. That only works if the product activates itself. Right now, every new customer requires a call with our team to get started. That's a ceiling on how fast we can grow and a tax on every school that wants to feed kids.

Q3 breaks that ceiling. Self-serve onboarding is the foundation every future distribution channel gets built on. A school that can onboard itself is a template for a food bank that can onboard itself. We build it once, correctly, for schools — and the pattern carries forward.

# 🔨 What We're Building

Q3 has four requirements and two stretch goals. Requirements are commitments — the quarter isn't a success without them. Stretch goals are the work that happens if requirements land ahead of schedule. Each item below describes what it is, why it's on the list, and what done looks like.

## Requirements

**1. Machine connection**

Before any other part of the product works, a machine has to be paired to an account. Right now this requires manual setup by our team. A self-serve pairing flow — account creation, machine discovery, connection confirmation — is the first screen a new customer hits and the technical foundation everything else builds on. Done when a new administrator can pair a machine end-to-end without our involvement.

**2. Batch scheduling**

The primary surface administrators use every week. They configure bread, spread, and jelly per day, set quantities, and set production times. The machine produces on schedule. Edge cases handled: holidays, early dismissals, day-of quantity overrides. Done when a school can run a full week of lunches from a single configuration session with no manual intervention from our team.

**3. Dietary configuration**

Required for any school with allergy policies — which is most of them. Administrators import a student roster, flag restrictions, and the system routes affected students to the correct machine profile automatically at schedule time. Without this, we cannot clear procurement at any school district that takes food safety seriously. Done when a school with nut-free students can run a full week without a manual workaround for affected students.

**4. Self-serve onboarding**

Wraps machine connection, batch scheduling, and dietary configuration into a guided first-run experience. Account creation, machine pairing, first schedule, confirmation. Done when a net-new administrator can go from zero to a scheduled production week in a single session, with no help from our team.

## Stretch Goals

**5. Parent visibility**

A read-only view of the daily lunch menu, shareable per school. No login required — parents open a link and see what's on the menu. Reduces inbound support questions and builds trust with the households the product ultimately serves. Done when a parent can view the current week's menu from a shareable link without logging in.

**6. Usage reporting**

A basic dashboard for administrators: sandwiches produced per day, waste, machine uptime. Gives administrators something to show their principal and gives us the operational data we need for support and iteration. Done when an administrator can pull a weekly production summary without asking us for it.

# 🗺️ Phases

The requirements have a natural build sequence — each piece depends on the one before it. Self-serve onboarding, for example, can't be written until machine connection, batch scheduling, and dietary configuration are all stable, because onboarding is a guided wrapper around those flows. The phases below reflect this dependency order. Stretch goals are sequenced after all requirements are complete.

## Phase 1: Machine Connection

```
1. Administrator creates account
2. App discovers machine on local network
3. Administrator selects their machine and confirms pairing
4. Machine is linked to account — scheduling interface unlocks
   → Machine not found on network = manual pairing code fallback
```

**Step 1: Administrator creates account.** Standard first-run screen — email, password, school name. No machine interaction yet. Account creation is a prerequisite for pairing; the machine needs an account to bind to before the connection can be established.

**Step 2: App discovers machine on local network.** After account creation, the app scans the local network for unpaired PB&J machines and surfaces what it finds. Discovery is passive — the administrator waits a few seconds while the scan runs. Most school networks allow local discovery; for those that don't, step 3 has a fallback.

**Step 3: Administrator selects their machine and confirms pairing.** The app lists discovered machines by model and serial number. The administrator selects theirs and confirms. If no machine was found in step 2, the administrator can enter a manual pairing code printed on the machine's underside — the fallback path for network environments that block local discovery, which is common in school IT setups.

**Step 4: Machine is linked to account.** A successful pairing binds the machine to the account and unlocks the scheduling interface. From this point on, the machine is associated with this school and cannot be paired to another account without an explicit unpair step.

## Phase 2: Batch Scheduling

```
1. Administrator opens weekly schedule grid
2. Configures active days — bread, spread, jelly, quantity, time
3. Marks holidays and early dismissal days
4. Confirms schedule — machine queues production jobs
5. Day-of: administrator adjusts quantity if needed
6. Machine produces on schedule
```

**Step 1: Administrator opens weekly schedule grid.** The default view is a seven-day grid for the current week. Each day is independently configurable. The administrator can also navigate forward to configure future weeks in advance — useful for setting up recurring schedules at the start of a school year.

**Step 2: Administrator configures active days.** For each active school day, the administrator sets bread type, spread, jelly flavor, quantity, and production time. A "copy to all days" shortcut handles the common case where every day has the same configuration. Individual days can still be overridden after copying.

**Step 3: Administrator marks holidays and early dismissal days.** Holidays are marked as inactive — the machine skips them automatically. Early dismissal days can have an adjusted quantity or an earlier production time set without reconfiguring the whole day.

**Step 4: Administrator confirms schedule.** Confirming the schedule queues production jobs on the machine. The machine stores the schedule locally so production fires at the configured time regardless of network connectivity. A confirmed schedule can still be edited — changes before the production window update the queued job; changes after are applied to the next occurrence.

**Step 5: Day-of quantity override.** On any given morning, an administrator can pull up the day's schedule and adjust the quantity before the production window opens. This exists because attendance is unpredictable. The override updates the queued job on the machine without touching the rest of the week's schedule.

**Step 6: Machine produces on schedule.** The machine works through its queued jobs at the configured times. No administrator action required unless something goes wrong.

## Phase 3: Dietary Configuration

```
1. Administrator imports student roster via CSV
2. Flags students with dietary restrictions
3. System maps restrictions to machine profiles
4. Administrator confirms weekly schedule — routing applied automatically
5. Machine produces correct profile for each student group
```

**Step 1: Administrator imports student roster.** The import accepts a standard CSV — name, grade, restriction type. The administrator exports this from whatever student information system the school uses; no reformatting required for common formats.

**Step 2: Administrator flags restrictions.** Restriction types are predefined: nut-free, gluten-free, or none. The administrator reviews the imported roster and corrects any mismatches before saving. Students without restrictions are unaffected by this configuration.

**Step 3: System maps restrictions to machine profiles.** Each restriction type maps to a machine profile defined at setup. Nut-free maps to a profile using sunflower seed spread and dedicated equipment that has never contacted peanuts. Gluten-free maps to a profile using gluten-free bread from a separate storage bay. Profiles are configured once and reused across all weeks.

**Step 4: Administrator confirms weekly schedule — routing applied automatically.** When the administrator confirms the weekly schedule in Phase 2, the system generates separate production queues per profile based on the restriction roster. The administrator does not take any extra steps — dietary routing is applied as a consequence of confirming the schedule.

**Step 5: Machine produces correct profile for each student group.** The machine works through production queues in sequence, switching profiles between runs. From the administrator's perspective, the right sandwich gets made for the right student automatically — no manual intervention required at production time.

## Phase 4: Self-Serve Onboarding

```
1. Administrator opens app for the first time
2. Creates account (Phase 1, Step 1)
3. Pairs machine (Phase 1, Steps 2–4)
4. Configures first weekly schedule (Phase 2, Steps 1–4)
5. Sets up dietary restrictions if needed (Phase 3, Steps 1–4)
6. Confirms — machine is ready to produce
   → Progress saved after each step; resume anytime
```

**Step 1: Administrator opens app for the first time.** First launch drops the administrator into the onboarding flow automatically. The flow is linear — each step must be completed before advancing. The goal is that a new administrator can complete onboarding in a single session without referring to documentation or contacting support.

**Steps 2–5: Guided execution of prior phases.** Onboarding doesn't introduce new functionality. It sequences machine connection, batch scheduling, and dietary configuration into a guided first-run experience with progress indicators, brief explanations at each step, and validation before advancing. An administrator who gets interrupted can close the app and resume exactly where they left off — progress is saved after each completed step.

**Step 6: Confirmation.** Once the schedule is confirmed and dietary configuration is complete (or skipped), the administrator sees a summary screen: machine paired, schedule set, first production window queued. The onboarding flow closes and the administrator lands in the main dashboard. From this point they are a fully operational customer.

Onboarding ships last in the sequence because it depends on every prior phase being stable. A guided flow is only as smooth as the underlying surfaces it walks the user through.

## Phase 5: Parent Visibility *(stretch)*

```
1. Administrator copies shareable menu link from dashboard
2. Shares link with parents (email, newsletter, etc.)
3. Parent opens link — sees current week's menu by day
4. Menu updates automatically as the schedule changes
   → No login required; link is school-specific and stable
```

**Step 1: Administrator copies shareable link.** The dashboard surfaces a per-school menu link in the settings panel. One tap copies it. The link is stable and permanent — it doesn't change when the schedule updates, so the administrator shares it once.

**Step 2: Administrator shares with parents.** Distribution is outside the app — the administrator pastes the link into their existing communication channels (email newsletter, school app, parent portal, etc.).

**Step 3: Parent views current week's menu.** The page shows the current week's menu by day: bread type, spread, jelly, and a generic note if dietary alternatives are being produced (e.g., "Nut-free option available"). Student names and individual restriction details are never exposed on this page.

**Step 4: Menu updates automatically.** When the administrator updates the weekly schedule, the parent page reflects the change immediately. No re-sharing required.

## Phase 6: Usage Reporting *(stretch)*

```
1. Administrator opens reporting dashboard
2. Selects current or past week
3. Views production summary — sandwiches produced, waste, uptime
4. Exports summary as CSV if needed
```

**Step 1: Administrator opens reporting dashboard.** The dashboard is a single screen accessible from the main navigation. The default view is the current week.

**Step 2: Administrator selects a week.** The administrator can step back through prior weeks using previous/next navigation. No date picker — week-by-week navigation is sufficient for the use cases this dashboard serves.

**Step 3: Administrator views production summary.** Three metrics per day: sandwiches produced, waste (scheduled but not produced, due to machine failure or manual cancellation), and machine uptime (percentage of scheduled production windows where the machine completed its run). Daily totals roll up to a weekly summary at the top of the screen.

**Step 4: Administrator exports summary.** A CSV download button produces one row per day with the same fields as the dashboard. Designed for administrators who need to drop the data into a spreadsheet for their principal or district office.
