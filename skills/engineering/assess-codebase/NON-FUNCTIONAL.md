# Non-functional properties

## Why this file exists

In a system built by engineers, planning covers two things: what it must do, and how it must hold up. In a system built fast with AI direction from a non-engineer, only the first is specified.

The brief is real and it is followed carefully. It says what screens exist, what fields they carry, what the workflow is, what the user needs to accomplish. It does not say what happens at fifty concurrent users, at ten million rows, when an import dies halfway, or when somebody who did not build it has to work out why a figure is wrong at 4pm on a Friday.

**So the functional requirements are usually met and the non-functional ones were never articulated.** The interface works. Whether the system works is a separate question that nobody asked.

That is why the screens can be genuinely good while the system underneath cannot be operated. It also predicts exactly where to look, because the absent properties are not randomly distributed. They are precisely the ones that never appear in a user-facing brief.

## The stance: prove it, do not assume it

With a conventionally built codebase you can assume a baseline and audit the exceptions. Someone chose the patterns, someone reviewed them, and the deviations are what need attention.

Here there is no baseline. Every practice is a hypothesis. Transactions, indexes, pagination, validation, error surfacing, authorisation on each path, consistent conventions: each is either present and demonstrable or it is absent, and the only way to tell is to check.

Do not infer a practice from finding it once. Consistency is the property in question. Finding a transaction proves a transaction exists, not that multi-step writes are transactional. Sample across modules, and prefer a ratio over an example.

Do not infer intent from a comment. A comment describing a design decision tells you a sentence was written.

## The six stresses

Everything in this file is one of six questions the original brief did not ask.

### 1. Concurrency: what happens when two people act at once

The single most under-tested property, because a person clicking through a demo never encounters it.

- **Transactions.** Count multi-step writes and how many run inside one. A sequence of independent awaits leaves half-applied state when the third one fails.
- **Read-modify-write.** Find `findUnique` followed by `update` on the same record. Without a version check or a lock, the second writer silently discards the first.
- **Generated identifiers.** Counting rows then adding one is a race. It is safe only where a unique constraint turns the collision into a visible error. Check which generators lack one.
- **Idempotency.** Every import, job and webhook will run twice eventually. Determine what happens: duplicate rows, doubled figures, or a clean no-op.
- **In-process guards.** A module-level boolean used as a lock works for exactly one server process and fails silently the moment there are two.
- **Cross-record invariants.** Where two records must agree, find out what enforces it under simultaneous edits.

### 2. In-flight work: what happens when the process is replaced

A distinct question from concurrency, and routinely missed. Deploys, restarts, crashes and container replacement all terminate work that is part-way through. **Ask what is in flight when the process dies, and what the user is told happened.**

- **Long user actions.** Recording, large uploads, multi-step wizards, anything running for minutes. Is progress held only in browser memory, or persisted somewhere it survives a reload. Absence of any client-side persistence, no IndexedDB and no local storage, means a tab crash loses everything.
- **Resumability.** Is an upload one request carrying the whole payload, or chunked with resume. A single large POST has no recovery position.
- **Silent failure on the client.** Find the catch around the upload. A discarded error means the user is shown success while the payload was lost, which is worse than an error message.
- **Server-side partial writes.** A stream written to disk before its database record is created leaves an orphan file, or a record with no file, depending on where it died. Neither is a transaction.
- **Background work started and not awaited.** Fire-and-forget promises that continue after the response do not survive process replacement, and usually leave partial state with no marker.
- **Deploy coordination.** Is there a `SIGTERM` handler, connection draining, or a maintenance mode that actually refuses traffic. A maintenance flag that only paints a status chip is not coordination.

Weight this by domain. For records, an interrupted write is an annoyance and the user re-enters it. For capture, it is permanent loss of something that cannot be recreated. See the domain table in `INTAKE.md`.

### 3. Time: what happens as data accumulates

- Tables that only grow: audit logs, events, snapshots, notifications. Is anything ever removed, and on what schedule.
- Queries with no limit and no filter, which get slower every month until they stop returning.
- Filters on unindexed columns, invisible at demo volume and fatal at production volume.
- Derived or cached data with no recomputation path, which drifts from source silently.
- Retention obligations for personal data, which are a compliance question and are never in a user brief.

### 4. Load: what happens with real users

- Connection pool bounds. An unbounded pool plus concurrent fan-out exhausts the database rather than queuing.
- Query counts per page. Multiply nested loops rather than reading each cap in isolation; individually bounded loops multiply into unbounded totals.
- Repeated identical queries within one request, and whether any request-level caching exists.
- Work done per request that should be precomputed.
- Memory per request: whole files buffered, whole tables loaded, whole spreadsheets parsed in process.

### 5. Real data: what happens when production data arrives

Demo data is clean, short, recent, and in one timezone. Production data is none of those.

- Field lengths. Default string columns are often much shorter than real business text, and the failure is at write time, during the import, in front of the customer.
- Encoding and special characters in names, addresses and free text.
- Dates at boundaries: financial year ends, daylight saving transitions, date-only values stored as timestamps.
- Null and zero where the demo always had a value, and whether the code distinguishes "zero" from "unknown".
- Volume during migration. A migration tested against an empty table behaves differently against ten years of rows.
- Records that cannot be matched back to their source system, which makes reconciliation impossible.

### 6. Diagnosability: what happens when a stranger has to find the problem

This is the property the original brief never contains, and it is what determines whether the system can be maintained at all.

Ask directly: **a figure is wrong, the person who built this is unavailable, where does someone start?**

- **Logging and error surfacing.** Are failures visible, or discarded into a console nothing captures. Count suppressed errors.
- **Type enforcement, actually running.** Not whether the language has types, but whether anything checks them, and whether escape hatches have hollowed them out.
- **Tests as executable specification.** Without them, nothing states what correct behaviour is, and a maintainer cannot distinguish a bug from an intention.
- **Consistent conventions.** Several ways of doing the same thing means a maintainer must learn all of them and cannot generalise from one module to the next.
- **Discoverable shared code.** If the abstraction cannot be found it will be rebuilt, so measure adoption ratios rather than existence.
- **Documentation that is true.** A wrong architecture document is worse than none, because a newcomer trusts it on day one.
- **Whether the system fits in a working context at all.** Measure the schema and the code against what a person can hold and what a tool can load. If the data model alone exceeds a working context, relationship-level mistakes are structurally likely for both, and no amount of care fixes it without external checks.

## Why this compounds

These properties are also the ones that decay fastest as a generated system grows, and for the same reason. Each is a global property. A transaction boundary spans several writes. Convention consistency spans modules. A shared component must be known about to be used.

Early on, when the system fits in view, they are handled. As it grows they are not, and the rate falls further the larger it gets. So the newest code, which is the least reviewed and often the least exercised, is also the least likely to have them.

**Weight recent work accordingly.** Sample the newest modules specifically rather than assuming they inherited the standards of the oldest.

## What follows for the recommendation

When these are absent, the remediation is not a list of fixes. Each of these properties has to become something a machine checks, because the whole reason they are missing is that no person held enough of the system to apply them consistently, and a new team will not hold it either.

That is the argument for automated enforcement over hiring, and it is the one that lands with people who assume more engineers is the answer.
