# Notebooks

A notebook is the record kept while executing a hands-on operation — provisioning infrastructure, running a migration, configuring a live service. It is written entry by entry as the work happens, holding the command as run, the output as returned, and what each one settled. Failures stay in it, because a failed attempt is the evidence for why the next one was shaped the way it was.

Every other document here holds what is true now. A notebook holds how it came to be true. That is what lets a plan or a standard stay short — the derivation has somewhere else to live.

## When to Open One

**A notebook covers one operation, not one ticket and not one phase.**

An operation is a continuous piece of work against one target system, with a state it starts from and a state it is done in. Standing up a server is an operation. Standing up the same thing for a second application is a second operation, whether or not it shares a ticket.

- **The test is a start and an end.** If you can say what was true before it began and what has to be true for it to be finished, it is a notebook. If you cannot, it is not an operation yet.
- **The target system separates two notebooks** more than the ticket does. The same procedure against a different system gets its own notebook, because its register describes different resources.
- **Phases are headings inside one notebook**, not separate files. Splitting an operation across files duplicates its register and its context, and neither copy stays current.
- **Not every ticket has one.** Writing code produces a commit and a PR description. A notebook exists where the commands *are* the work and the output is the only record of what happened.

The procedure for running one — when to open it, what to write when, when to close it — is [`procedures/sop_notebook.md`](../../../procedures/sop_notebook.md).

## Two Zones

A notebook is two documents in one file, with opposite rules, and the entry marker is the boundary between them.

**The head is current state.** The resource register, the context, and the entry index. Rewritten freely — the register is the table you read when you want to know what exists, not a history of what was created.

**The body is the record.** Append-only. New entries go in at the `<!-- NEXT ENTRY -->` marker at the bottom and are never touched again. A correction is a new entry that names the one it corrects.

Having exactly one legal edit point in the body is what makes a violation visible: an entry rewritten in place happened somewhere other than the marker.

## Entry Types

Every entry carries a number, a type marker, and a title stating what it did. The type is decided by what the entry *is*, not by what it looks like.

**🔧 Action** — a command that changes something. Intent, Command, Output, Settled.

**🔍 Observation** — a read that establishes state. Same four fields, but nothing changed and the register is not touched. Worth an entry when its answer shapes a later decision.

**⚖️ Decision** — a choice between options, with no command at all. Intent, Options, Decision, Why, Consequence. Forcing command-and-output onto one of these is what makes it unwritable.

**📄 File** — a file written into a repository rather than a resource created in a system. Intent, Path, Shape, Settled. The file is the source of truth for its own contents and is not duplicated in the notebook.

The markers are what make a notebook extractable. Decision entries are where a standard comes from; the marker is what finds them across a whole operation without reading every entry that was scaffolding.

## Conventions

- **Output goes in verbatim.** Trim only noise that carries nothing. Never round a number and never summarize a result — figures that look incidental are what a later read divides, counts, and reasons from.
- **A failed entry stays**, marked in its title and superseded by a later one rather than deleted.
- **Record what a value was when it was pasted forward.** A `# ->` comment carries what came back, so the sequence can be re-run.
- **State the prediction before the result.** An entry that says what it expects and then shows it is evidence. An entry that only shows the result is a transcript.
- **Name what an entry does not prove**, and where the real check was recorded.
- **No secret is ever written into a notebook.** Record the parameter name and the fact that it was set.
- **A console action says so.** It has no command to record, so the entry names the screen and the fields.

## Examples

| File | What It Covers |
|---|---|
| `nb_order-service-provisioning.md` | Provisioning — putting the order service on a dedicated host at a stable hostname over TLS, with a deploy workflow that can replace the running process. All four entry types, two failures and their supersessions |
| `nb_spread-registry-backfill.md` | Migration — filling a table from a config file and moving reads onto it under live traffic. A phased notebook: rehearsal, backfill, cutover. Shows a gate that fails, and a register that holds data and flags rather than infrastructure |
