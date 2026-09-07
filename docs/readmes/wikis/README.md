# Wiki READMEs

A wiki README is the front door of a documentation directory. It governs sibling documents rather than code or work, and its reader has arrived at a directory of standards or reference material and needs to know what is in it and which file answers their question.

It does three jobs. It says what the directory governs and when to read it. It routes to the children, including the ones that do not exist yet. And it holds the small number of things a reader needs that no child file owns. That third job is what makes it a document rather than a table of contents, and it is also the one that has to be defended, because it is the only part of the file with no natural size limit.

## When to Use One

**A documentation directory gets one as soon as it holds more than one or two files.**

Below that, a reader can open both files faster than they can read about them. Above it, the directory is a place someone lands without knowing what is in it.

- **The test is whether a reader can be sent to the directory instead of a file.** If the answer to "where do I find out how we do X" is a directory name, that directory needs a front door.
- **It is written for whoever reads the children.** A wiki of coding standards is read mostly by LLM sessions, so its README is too — but it has to be legible to a person, because a person is who checks it is right.
- **A nested directory gets its own.** A category holding type directories has a README, and so does each type directory. The parent routes to the children; it does not describe their contents.
- **Not to be confused with a repo README.** A repo README's reader wants to run something. This reader wants to know a rule before they write something.

## Conventions

### Structure

Two sections are fixed and come first, in this order. Everything after them is optional and depends on what the directory holds.

1. **Summary** — what the directory governs, who it is for, and when to open it. Prose, one or two paragraphs.
2. **What's Here** — the index.
3. **Context** *(optional)* — the lookup values the children share.
4. **Good to Know** *(optional)* — the facts that belong to no child.

**The index comes second because a reader who lands on a directory wants to know what is in it.** Anything placed above the index is content a reader has to get through before they can find the file they came for.

**The title block is `Created` and `Updated`, and nothing else.** What the directory applies to and when to read it belong in the summary, where the reader is already reading, rather than in a metadata line they skim past.

**The index is a two-column table**: the file or directory, and what it covers. One row each.

**Unwritten children are listed and not linked**, marked `*(planned)*`. That is how the intended shape of the directory stays visible without pretending a file exists.

### The README does not hold the rules

**A rule lives in the document that proves it.** The temptation is to write the shared model at the top of the README, above the index, where every reader gets it for free. Resist it — a README that holds content accumulates it, and there is no line to stop at. Every rule a child rejects as too specific bubbles upward, and the README becomes the document nobody maintains and everybody skims.

**A rule is lifted into the README only when two children independently prove it.** A model derived from one child is that child's document written at a higher altitude, and it will be wrong in ways nobody catches until the second child arrives and disagrees.

**Boilerplate that is true of the whole collection is stated once at the collection root**, not repeated in every directory README. A reader who has to be told the same convention in five places learns to skip that section in all five.

### Context

**A `Context` block holds lookup values the children share** — identifiers, hostnames, table names, expected counts, units. The same convention as a plan's phase context: things a reader would copy rather than read.

**The test is whether you would copy-paste it.** If it explains *why*, it is not context — it is either a rule, which belongs in a child, or a fact nobody owns, which belongs below.

**Most standards directories have no context at all**, because a standard that names a real value is not a standard. Omit the section rather than filling it. It earns its place in a project wiki, where the children genuinely share environments, endpoints, and identifiers.

### Good to Know

**This section holds only what no child owns** — a constraint the whole directory rests on, a shared resource, a trap that lives in the gap between two documents.

**The test is whether a child could hold it instead.** If one could, it goes there. A fact stated here and again in the child that governs it is a fact with two places to go stale, and the child is the one a session actually loads.

**Keep it to a handful of entries.** A long one means rules are being written here rather than in the children, which is the failure this whole section exists to prevent.

## Examples

| File | What It Covers |
|---|---|
| `rdm-wiki_spread-standards.md` | A leaf directory — four child standards, a `Context` block of shared units and identifiers, and one cross-cutting trap that lives between two of the children |
| `rdm-wiki_machine-control.md` | A parent directory — routing to nested type directories rather than files, with a planned child and no `Context`, because nothing its children share is a lookup value |
