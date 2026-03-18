# SOP: Write PR Description

**Purpose:** Produce a PR description in `prd_` format and post it directly to GitHub. Triggered after a ticket is complete and a PR has been created. No writing from scratch.

---

## Trigger

Drop a PR link in chat.

Load context and begin Phase 1.

---

## Phase 1 — Load Context

Fetch the following:

1. **PR files changed** — via GitHub MCP (`pull_request_read`, method: `get_files`)
2. **PR diff** — via GitHub MCP (`pull_request_read`, method: `get_diff`)
3. **Ticket** — already in session context, or re-fetched via your project management tool using the ticket ID from the branch name or conversation
4. **Feature spec** — if one exists for this feature, find it in the repo and read it before writing
5. **`prd_` format example** — find the `prd_` example for the artisan spread selection feature in this repo and use it as your format reference

No input needed at this step.

---

## Phase 2 — Write and Post

Using the loaded context, produce the PR description with three sections:

### Summary
One paragraph. What changed and why — merged. Code-level detail, not user journey. Assumes the reviewer knows the ticket but not the implementation choices. Reference the ticket number.

### Current vs. New Behavior
Two labeled code blocks showing the before and after flow from the user or developer's perspective. Each step is a single action. The diff between the two blocks is where the reviewer's attention should go.

### How to Test
Exact commands with expected outputs. What a passing result looks like. Specific enough that the reviewer can verify without asking the author.

Once written, post the description and update the title via GitHub MCP (`update_pull_request`). Title follows conventional commit format (`feat:`, `fix:`, `refactor:`, etc.), all lowercase, matching the ticket.

Confirm in chat: `"PR #[N] updated — title and description posted."`

> ⏸️ **GATE → human review.** Open the PR on GitHub. Read the description and edit directly if anything needs adjusting. When it looks right, merge.

---

## Notes

- If the PR covers multiple tickets, the Summary should cover all of them and the Current vs. New Behavior should show the combined change.
- How to Test may require iteration — if the steps aren't quite right after review, correct them in the PR directly rather than re-running the procedure.
- This procedure assumes full session context from the work session. If context is missing, provide the ticket number alongside the PR link before triggering.

---

## Example

The following is a finished PR description produced by this procedure, using the artisan spread selection feature as the example.

---

# feat: add artisan spread selection to sandwich ordering flow

**Ticket:** ENG-312  
**PR:** #47  
**Reviewer:** Sam Torres

## Summary

Adds `SpreadSelector` as the third step in the sandwich ordering flow, between `JellySelector` and the confirmation screen. Previously the system hardcoded peanut butter with no option to change it — this was the highest-voted item in the Q1 feedback survey. Three spreads are now supported: `peanut_butter`, `almond_butter`, and `sunflower_butter`. Selection persists across back-navigation using the same shared state pattern as `BreadSelector` and `JellySelector`. The ordering API now accepts `spreadType` as a required field.

## Current vs. New Behavior

```
Current:

  User opens PB&J Builder
  → Selects bread type
  → Selects jelly flavor
  → Confirms order
  → System assembles with peanut butter (hardcoded, no choice offered)

New:

  User opens PB&J Builder
  → Selects bread type
  → Selects jelly flavor
  → Selects spread from available options        ← new step
  → Confirms order
  → System assembles with chosen spread
```

## How to Test

```bash
# Start the simulator
npm run simulator

# Run the order wizard
npm run dev order
```

Walk through the full ordering flow. After selecting jelly, the spread step should appear with two visible options (`peanut_butter` and `almond_butter`). Select one and continue to confirmation — the selection should appear in the summary. Navigate back to the jelly step and forward again — the spread selection should still be set.

To verify the API validation:

```bash
# Submit an order without a spread — should return 400
curl -X POST http://localhost:3000/orders \
  -H "Content-Type: application/json" \
  -d '{ "breadType": "sourdough", "jellyFlavor": "grape" }'

# Expected: 400 { "error": "spreadType is required" }
```

```bash
# Run the full test suite
npm test
```

New tests are in `SpreadSelector.test.tsx` and `order.handler.test.ts`. All existing tests should pass unchanged.
