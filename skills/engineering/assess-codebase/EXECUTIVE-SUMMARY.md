# Executive summary

A separate document, or the first page, written for people who will never read the technical report.

**They are not deciding whether the code is good. They are deciding whether the business loses money, breaks a commitment, or fails an audit.** A finding that cannot be expressed in those terms does not belong here, however serious it is technically.

## Format

- **The recommendation, in the first sentence.** Go, no-go, or go under stated conditions.
- **Three reasons. Not five.** Three is what survives a meeting and what gets repeated afterwards. If you have eight findings, they group into three consequences.
- **Each reason: what it costs, how long, how likely.** Money and time first, in that order.
- **A cost and a timeframe** for the recommended path.
- **What is already working,** briefly. Without it the document reads as a case for the prosecution and gets discounted.
- **No technical vocabulary.** No file names, no counts of lines or tables, no framework names.

One page. If it runs longer, the reasons are not yet reduced to consequences.

## Group by consequence, then translate

Two steps, and the first is the one usually skipped.

**Group first.** Findings that cost the business the same thing are one reason, whatever their technical cause. A calculation defect, an unreconciled second source of the same figure, and a derived table nobody checks are three findings and one consequence: the numbers cannot be relied on. Merge them. The technical report keeps them separate; the summary does not.

**Then translate into money and time.** Management is not deciding whether a backup system exists. They are deciding what an outage costs, how long it lasts, and how likely it is. A reason that names a mechanism has not finished being translated.

| Still technical | Translated |
|---|---|
| There are no working backups | If the server fails we lose every commercial record since the last manual export, and the team stops for a day while it is rebuilt from whatever people have locally |
| The evidence that we delivered is unverifiable | We cannot defend a billing dispute with a customer, so a contested invoice is written off rather than argued |
| Unenforced references, no transactions | Revenue quietly leaves the reports during ordinary work, so the figures drift from reality with nobody able to say by how much |
| No error tracking, no tests, no CI | Every fix takes longer and carries a chance of breaking something else, so the running cost is permanently higher than it looks |

Notice what disappears: backups, transactions, tests, references. All real, all in the technical report, none of them something a board discusses.

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

> **Recommendation: do not retire the current stock system yet. Run both in parallel for one quarter, then decide. Three reasons.**
>
> **1. We cannot trust the stock figures, and we cannot tell how wrong they are.** Stock is stored as a running number rather than built up from recorded movements, and nothing checks it against a physical count. Once it disagrees with the shelf there is no way to reconstruct why. Cost: unknown until we run a full count, which is a day of warehouse time and the first thing we would do. Likelihood it is already wrong: high, and we have no way to prove otherwise.
>
> **2. If the server fails we lose the stock position and stop despatching for a day.** The system reports that backups are working. Nothing is being written. Recovery would mean rebuilding the position by hand from purchase orders and despatch notes, which is a day at best with the warehouse idle. Likelihood in any given year: moderate, and it is the kind of risk that costs nothing to remove now and everything to discover later.
>
> **3. It costs more to run than it looks, and that does not go away.** Every change carries a chance of breaking something else because nothing checks the work before it ships, so ordinary requests take weeks rather than days. On current form that is roughly one full-time person's cost absorbed into slower delivery, permanently, unless the foundations are put in.
>
> **What is already working:** the picking and despatch workflow is well designed and staff prefer it to the current system, access controls are consistent, and the business rules are documented well enough for another team to pick up.
>
> **Recommended path:** two engineers, six to nine months, to a position we would rely on. The first month removes reason 2 and costs very little.
>
> **If we do nothing:** we keep committing stock we may not have, with no way to recover the position if the server fails.

Note what the example does. Three reasons, not four or five. Each opens with a cost and a duration, not a mechanism. Reason 3 is the whole maintainability half of the report compressed into an annual running cost, because that is the only form in which it competes for budget. Likelihoods are stated honestly rather than inflated, which is what makes the rest credible. The closing line usually decides the meeting.
