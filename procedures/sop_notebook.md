# SOP: Notebook

**Created:** 2026-09-07
**Purpose:** Run a notebook — the record kept while executing a hands-on operation, written entry by entry as the work happens. Covers when a notebook is opened, what each entry holds, how it is appended to, and when it is closed. The notebook is later read as evidence: to reproduce a step, to establish what state something is in, or to extract a standard from what was settled.

---

## Trigger

A hands-on operation is about to begin — provisioning infrastructure, running a migration, configuring an external service, or any sequence of commands against a live system that has a starting state and a defined end.

Claude begins Phase 1.

**Not a notebook.** Writing code, reviewing a document, or planning work. Those produce a commit, a review, or a plan. A notebook exists where the commands are the work and the output is the only record of what happened.

---

## The Unit

**A notebook covers one operation, not one ticket and not one phase.**

An operation is a continuous piece of work against one target system, with a state it starts from and a state it is done in. Standing up a server is an operation. Standing up the same thing for a second application is a second operation, whether or not it shares a ticket.

- **The test is a start and an end.** If you can say what was true before it began and what has to be true for it to be finished, it is a notebook. If you cannot, it is not an operation yet.
- **The target system separates two notebooks** more than the ticket does. The same procedure against a different system is a new notebook, because its register describes different resources.
- **A ticket may have several notebooks, and a notebook may outlast its ticket.** The ticket workspace records the arc of the work. The notebook records the execution of one operation inside it.
- **Phases are headings inside one notebook**, not separate files. Splitting an operation across files duplicates its register and its context, and neither copy stays current.

---

## Phase 1 — Open the Notebook

Create the file before running the first command. A notebook written after the fact is a summary, and the thing it loses is exactly the thing it exists for.

**Name it after the operation:** `nb_{operation}.md` — `nb_v2-server-provisioning.md`, not the ticket number and not the phase.

Write the skeleton and nothing else:

```markdown
# Notebook — {Operation}

**Ticket:** {id}
**Started:** {YYYY-MM-DD}
**Status:** 🟨 Open
**Purpose:** {What this operation does and what it is done when. One or two sentences.}

## 📊 Resource Register

Nothing yet.

## 🔭 Context

{Preconditions inherited from outside this notebook that shape what the commands
 here can look like. Omit the section if there are none.}

## 🧪 Entries

<!-- NEXT ENTRY -->
```

**Context holds only what came from outside this notebook.** Another notebook, an earlier ticket, a prior audit. Facts established by this notebook's own entries are never restated here — they are above, in the entries.

**The test is whether a fact constrains a command below it.** If it does not, it is background and belongs in the ticket workspace. Context is also where an inherited assumption can be *disproven*: it lives in the mutable head, so when an entry proves one false, the entry records it and the head is corrected.

> ⏸️ **GATE → User.** The purpose sentence and the operation's end state are agreed before the first command runs.

---

## Phase 2 — The Loop

This is the operating procedure for the whole body of the work, and it repeats until the operation is done.

1. **Propose one command, with its intent.** Say what the command is for and what its result will settle before showing it. One command per turn — a batch that fails somewhere in the middle produces output that cannot be attributed.
2. **The user runs it and pastes the output back.** Commands are run by the user, not by Claude. That is what makes the output a record of the real system rather than a sandbox.
3. **Write the entry immediately, in the same turn the output arrives.** Not at the end of the phase and not at offboarding. An entry written later is written from memory, and the verbatim output is already gone.
4. **Update the Resource Register if anything was created, changed, or destroyed.** Same turn.

**Say that the entry was written.** The user cannot see the file, and a record they have to ask about is a record they have to audit. One line naming the entry number and title is enough.

**Append at the marker.** The body of the notebook has exactly one legal edit point: `<!-- NEXT ENTRY -->`. Replace it with the new entry followed by the same marker, so it returns to the bottom. Never rewrite the file to append to it.

**Nothing in the body is edited after it is written.** A correction is a new entry that names the one it corrects.

**Where an operation runs in phases**, each phase is a `## 🧪 Phase N — {Name}` section replacing the single `## 🧪 Entries` heading, with a date and one sentence saying what the phase does. Entries number continuously across the whole notebook rather than restarting, so an entry has one address for the life of the operation.

---

## Phase 3 — Writing an Entry

Every entry carries a number, a type marker, and a title stating what it did.

```markdown
### 007 🔧 Write the nginx server blocks
```

Four entry types, each with fixed fields. The type is decided by what the entry is, not by what it looks like.

### 🔧 Action — a command that changes something

**Intent.** What the command is for, and what it will settle.
**Command.** As run.
**Output.** As returned.
**Settled.** What is now true that was not before.

### 🔍 Observation — a read that establishes state

Same four fields. The difference is that nothing changed, and the register is not touched.

An observation is worth an entry when its answer shapes a later decision. Reading state you already know does not need recording.

### ⚖️ Decision — a choice between options, with no command

**Intent.** The question being settled.
**Options.** What was actually available, including the one not taken.
**Decision.** The choice.
**Why.** The reason it wins. One paragraph.
**Consequence.** What this obliges later, or what it makes impossible.

**A decision entry has no command and no output, and forcing those fields onto it is what makes it unwritable.** These are the entries a standard is later extracted from — the marker is what makes them findable across a whole notebook.

### 📄 File — a file written into a repository rather than a resource created in a system

**Intent.** What the file is for.
**Path.** Where it lives.
**Shape.** The choices made in it and why. A table where there are several.
**Settled.** What now exists.

**The file is the source of truth for its own contents and is not duplicated here in full.** The notebook records why it took the shape it did.

### Rules for every entry

- **Output goes in verbatim.** Trim only noise that carries nothing — a package manager's progress bars, a donation banner. Never round a number and never summarize a result. Figures that look incidental are what a later read divides, counts, and reasons from.
- **A failed entry stays**, marked in its title. The failure is usually the evidence for why the next attempt was shaped the way it was.
- **A correction is a new entry.** Title it as correcting the entry it corrects, and leave the original in place.
- **Record what a value was when it was pasted forward.** Where a value is read from one command and used in the next, a `# ->` comment carries what came back, so the sequence can be re-run.
- **State the prediction before the result** where there is one. An entry that says what it expects and then shows it is evidence; an entry that only shows the result is a transcript.
- **Name what an entry does not prove.** A check that will only become load-bearing later says so, and says where it was recorded.
- **No secret is ever written into a notebook.** Not a connection string, not a key, not a password — the notebook is the most-read artifact the operation produces. Record the parameter name and the fact that it was set. A command whose arguments contain a secret is recorded with the value replaced.
- **A console action says so.** It has no command to record, so the entry names the screen and the fields.

---

## Phase 4 — Close the Notebook

A notebook closes when the operation reaches the end state agreed in Phase 1 — usually before its ticket does.

- **Final pass on the Resource Register.** It is the current state of what exists, not a history of what was created. An entry that says a resource returns 502 until a later step is stale the moment that step runs.
- **Write the Entry Index**, under the register. Number, marker, and title, in order. It is written once at close rather than maintained per entry, because maintaining it costs an edit at the top of the file for every append — and because by the end you know which entries mattered.
- **Set the status** to ✅ Complete with the date.
- **Leave `<!-- NEXT ENTRY -->` in place.** A closed notebook that turns out to need one more entry gets one, appended like any other.

---

## Principles

**The notebook is written during, or it is not a notebook.** Everything the format is for — verbatim output, retained failures, the reasoning that was live at the time — is unavailable an hour later.

**Two zones, two rules.** The head is current state and is rewritten freely: register, context, index. The body is the record and is append-only. One marker separates them, which is what makes a violation visible.

**Facts in, reasoning out — except here.** Standing documents hold what is true now. The notebook is the counterpart that holds how it came to be true, which is why a plan or a standard can stay short.

**The reader was not there.** Every entry is read by someone with no memory of the session. The intent field is what carries them, and it is the field most likely to feel unnecessary while writing.

**It is a source, not an archive.** A notebook is read to reproduce a step, to establish current state, or to extract a standard from what was settled. Write every entry as something that will be quoted.
