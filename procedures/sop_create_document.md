# Document Creation and Review Procedure

**Created:** 2026-06-24
**Purpose:** Create new documentation or review and update existing documentation within a project wiki. Covers identifying the right document type, writing from a PB&J example first, iterating on format and content, and saving the final doc to the correct location.

---

## Trigger

User says: `"let's write a [doc type] for [project]"` or `"review the docs in [directory]"` or `"we need documentation for [topic]"` — or any variation indicating they want to create, update, or review a document.

Claude begins Phase 1.

---

## Phase 1 — Orient to the Task

Before writing anything, establish three things:

1. **What kind of document is this?** Match the task to a known doc type from the documentation examples index. If the doc type exists, find and read the example before proceeding. If it doesn't exist, a new example needs to be created first — see Phase 2A.

2. **Where does it live?** Identify the correct directory. New documentation for a project goes under `projects/[project-name]/wikis/[category]/`. New examples go under `plugins/documentation-examples/[category]/`.

3. **Is there existing documentation to review?** If the user is working in an existing directory, list it and read anything relevant before proceeding. Don't write without knowing what's already there.

> ⏸️ **GATE → User.** Surface what you found — the doc type, the target location, and any existing docs worth knowing about. Confirm the direction before writing anything.

---

## Phase 2A — New Doc Type: Write the Example First

If no example exists for the requested doc type, create it for PB&J Machine Co. before applying it to the real project. This separates format decisions from content decisions — the user can focus on structure and style without getting distracted by real project details.

**Write the PB&J example:**
- Use the PB&J Machine Co. fictional context throughout
- Apply the formatting conventions from existing examples: H1s with emojis for main sections, narrative prose over bullet lists, no `---` dividers
- Make the content realistic enough to evaluate but generic enough that feedback is about format, not facts

Save the example to `plugins/documentation-examples/[category]/[prefix]_pbj-co.md`.

> ⏸️ **GATE → User.** Ask the user to review the example. Iterate on format, section names, section order, tone, and level of detail until the structure is locked. Do not apply to the real project until the example is approved.

**After the example is approved:**
- Update `plugins/documentation-examples/[category]/README.md` to add the new doc type with status ✅
- Update `plugins/documentation-examples/README.md` to add the new prefix to the doc type table

---

## Phase 2B — Existing Doc Type: Read the Example

If the doc type already exists, read the example before writing. The example is the source of truth for structure, tone, and formatting conventions. Don't rely on memory — read it fresh.

---

## Phase 3 — Write or Update the Document

With the doc type confirmed and the example read, write the document for the real project.

**For new documents:**
- Follow the structure of the example exactly — same section names, same heading levels, same formatting conventions
- Write for durability. Avoid references that are specific to today's conversation — if a sentence only makes sense because of something said in this session, it probably doesn't belong in the doc
- Be direct. Cut anything that doesn't add information. Fluff makes documents harder to load as context in future sessions

**For existing documents:**
- Read the current version before making any changes
- Make targeted edits rather than full rewrites unless the doc needs a structural overhaul
- If the doc was written before the current formatting conventions were established, align it to the current style as part of the update

Save to the correct location and confirm the path.

> ⏸️ **GATE → User.** Ask the user to review the document. Iterate section by section if needed. The document is not done until the user explicitly approves it.

---

## Phase 4 — Update Indexes and READMEs

After the document is approved, check whether any index files need updating:

- If a new doc was added to a wiki, check whether the wiki's `README.md` has a table of contents and add the new entry
- If a new doc type was created, confirm Phase 2A cleanup (prefix table + category README) was completed
- If an existing doc was significantly restructured, update its entry in any README that describes it

Surface any index updates to the user before saving.

---

## Principles

**Example first, project second.** When creating a new doc type, always write the PB&J example before touching the real project. The separation keeps format feedback clean and builds a reusable template at the same time.

**Read before writing.** Always read the relevant example and any existing docs in the target directory before producing output. Writing without reading leads to duplication, format drift, and docs that don't fit the surrounding context.

**Durability over completeness.** A document that loads cleanly in a future session and gives accurate context is worth more than a document that captures every nuance of today's conversation. When in doubt, cut the detail that only makes sense today.

**One approval gate per document.** Don't move to the next document until the current one is explicitly approved. Stacking documents without review leads to format drift that compounds across the session.

**Narrative top to bottom.** Documents should read as a clear, linear story from first section to last — each section following naturally from the one before it. Avoid structures that require the reader to jump around. A document that flows is a document that gets read.

**Generic over specific.** Write at the level of the concept, not the current conversation. Avoid naming specific features, tickets, or decisions that are only relevant today. The document should be as useful in six months as it is now.

**Cut the fluff.** Every sentence should earn its place. Remove preamble, redundant restatement, and anything that explains what the reader is about to read rather than just saying it. Tighter docs load faster as context and are easier to maintain.
