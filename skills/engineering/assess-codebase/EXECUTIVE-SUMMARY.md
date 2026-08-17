# Executive summary

A separate document, or the first page, written for people who will never read the technical report.

**They are not deciding whether the code is good. They are deciding whether the business loses money, breaks a commitment, or fails an audit.** A finding that cannot be expressed in those terms does not belong here, however serious it is technically.

## Format

- **The recommendation, in the first sentence.** Go, no-go, or go under stated conditions.
- **At most five reasons.** Five is the ceiling, not the target. Three strong reasons beat five padded ones.
- **Each reason: what happens, how likely, what it costs.** In that order.
- **A cost and a timeframe** for the recommended path.
- **What is already working,** briefly. Without it the document reads as a case for the prosecution and gets discounted.
- **No technical vocabulary.** No file names, no counts of lines or tables, no framework names.

One page. If it runs longer, the reasons are not yet reduced to consequences.

## Translation

Every reason is a consequence, not a condition. The technical statement is the evidence; it belongs in the technical report and is referenced, not repeated.

The pattern, with illustrative phrasings rather than findings from any particular system:

| Technical finding | Shape of what management is told |
|---|---|
| A calculation returns a constant where it should vary | The figures on that dashboard have been wrong by a large factor for as long as the system has existed. Any decision taken on them was taken on a number that was not real. |
| Backup writes nothing; verification inspects its own metadata | The system reports that backups are working. There are none. If the server fails we lose everything since somebody last exported by hand. |
| Unenforced references, no transactions, discarded errors | Records can detach from their parent during routine operations, with no error and no log. The value attached to them stops appearing in reports and nobody is told. |
| No change record, hand-written database patches present | We cannot demonstrate who changed what, or when. For a system feeding financial reporting, that is a question for our auditors rather than for IT. |
| No tests, no automated checking, no team, codebase exceeds working memory | We cannot safely fix any of the above. Changes cannot be verified before release, and nobody currently holds the system in their head. |
| No rate limiting, no central identity, sessions outlive account disablement | A departing employee keeps access for the rest of the day, and passwords can be guessed without limit. |
| No shared component library, screens built individually | Ordinary interface changes cost weeks rather than days, permanently. |

Notice what drops out: architecture, naming, duplication, type safety, test coverage as a number. All are real, all belong in the technical report, none survives contact with the question "what does this cost us".

## Quantify where you honestly can

Prefer a stated range with its basis to a vague adjective.

- "Data loss exposure is currently the period since the last manual export, typically weeks."
- "Recovery from a server failure would take a full day of manual work, during which the system is unavailable."
- "The repair programme is three engineers for nine to fifteen months."

Where a number is not knowable, say what it depends on rather than inventing one: "The cost depends on how many records are already inconsistent, which a two-day survey would establish."

Never present a modelled figure as a measurement.

## Order by exposure, not by technical severity

Lead with whatever loses the most money or creates the most obligation. A wrong number acted on by executives usually outranks a security gap that has not been exploited, because one is already happening and the other is contingent.

Calibrate to the tier from `INTAKE.md`. For a Tier 3 informational system there may be no revenue consequence at all, and the honest executive summary is short and says so: the risks are privacy and staff time, the system is fit for its purpose, proceed.

## Give them the decision, not the dilemma

Management needs a recommended action with conditions, not a list of concerns to adjudicate. Close with:

- What to do now.
- What must be true before the next step.
- What it costs and how long, stated as calendar rather than effort, since hiring lead time is real.
- What happens to the people currently using it, and to whoever built it.
- What happens if nothing is done.

The last two are frequently omitted. The final one is often the most persuasive, and the one before it usually determines whether the recommendation is followed at all. A report that reads as a judgement on a person rather than on a system gets resisted regardless of its evidence, and in systems built this way the builder is commonly the product owner and the only holder of the domain knowledge. Say what their role becomes.

## Worked example

Illustrative and synthetic. A warehouse stock and despatch system, assessed as Tier 1 because nothing else holds the stock position.

> **Recommendation: do not retire the current stock system yet. Running both in parallel for one quarter is reasonable, subject to four conditions.**
>
> **1. The stock figures drift and nobody is told.** Stock levels are stored as a single running number rather than derived from recorded movements, so once a count disagrees with reality there is no way to reconstruct how it got there. We found no reconciliation against the physical count. Cost: unknown until a full count is run, which is the first thing we would do.
>
> **2. There are no backups, and the system reports that there are.** The administration screen shows completed and verified backups. No data is being written to any backup location. If the server fails today we lose everything since somebody last exported by hand, which nobody could tell us the date of.
>
> **3. Two despatches at the same moment can both take the last unit.** The check for available stock and the commitment of that stock are separate steps with nothing preventing another order in between. At current volumes this is rare. At the volumes planned for next year it stops being rare.
>
> **4. We cannot safely fix any of the above.** There is no automated checking of changes and no test coverage, so every repair ships unverified, and the person who built the system is the only one who understands it.
>
> **What is already working:** the picking and despatch workflow is well designed and staff prefer it to the current system, access controls are applied consistently, and the business rules are documented well enough for a new team to pick up.
>
> **Recommended path:** two engineers, six to nine months, to a position we would be comfortable relying on. The first month is backups and monitoring, which is inexpensive and makes everything after it verifiable.
>
> **If we do nothing:** we continue committing stock we may not have, with no way to recover the position if the server fails.

Note what the example does and does not do. Every reason is a business consequence. None names a file, a table, a framework or a count. Reason 3 states the current likelihood honestly rather than inflating it, which is what makes the rest credible. The strengths paragraph is specific enough to be believed. The closing line is the one that usually decides the meeting.
