# Verification

The strictness rules. Follow these literally. Every published error in the assessment this skill was built from would have been caught by one of them.

## The rule

**No figure, ratio, count or quoted line reaches the output unless you ran the measurement yourself in this session and can restate the command.**

Not "an audit found". Not "the analysis showed". You ran it, you saw the output, you can reproduce it on demand.

If you cannot meet that bar for a claim, you have three options and hedging is not among them:

1. Run the measurement now.
2. Drop the claim.
3. Publish it explicitly marked as unverified, naming what would settle it.

## Subagent contract

Subagents are for **discovery and hypothesis**, never for figures.

**They may return:** file paths, function names, patterns worth investigating, candidate findings, an argued case, a claim that something came back clean.

**They may not supply, and you may not copy:** any number, ratio, percentage, count, or quoted code line that appears in the output.

Treat every subagent claim as a lead requiring independent confirmation. Word each brief accordingly: ask for locations and reasoning, and state that all quantities will be re-measured.

When a subagent reports a striking number, the correct response is to run it yourself before believing it, in either direction. Subagents both overstate and understate. In the source assessment, one understated a query-count finding and four overstated other figures.

## The claim ledger

Keep a running ledger while working. Nothing enters a deliverable at status `claimed`.

```
CLAIM                                  | COMMAND                              | RESULT      | STATUS
812 of 940 FK columns unconstrained    | python parse of schema.prisma        | 812 / 940   | verified
periodElapsed always returns 0.05      | node arithmetic repro                | 0.05        | verified
scripts were run against production    | (none available)                     | -           | unverifiable
```

Three statuses only:

- **verified**: you ran it, output matches, command restatable.
- **unverifiable**: no evidence obtainable statically. Publish only with the limit stated inline and the test that would settle it.
- **withdrawn**: could not be substantiated. Record it; the withdrawn list goes in the report.

Anything else does not ship.

## The coverage ledger

The claim ledger keeps individual figures honest. It does nothing about the larger failure, which is silently not covering a dimension at all and producing a report that reads as complete.

Do not solve this by checking at the end from memory. That is the same defect this skill diagnoses in codebases: a global property maintained by somebody remembering. **Make coverage an artifact, maintain it during, and publish it.**

Keep a second ledger alongside the first, one row per dimension:

```
DIMENSION              | STATUS         | EVIDENCE / REASON
Security               | covered        | 15 findings, 5 verified blockers
Data integrity         | covered        | 812/940 unconstrained, measured
Concurrency            | covered        | clean: transactions present at 12/12 sampled sites
Supply chain           | not applicable | Tier 3, no external dependencies of consequence
Frontend               | not done       | out of scope this pass, cost-of-ownership question deferred
```

Three states, and the third is the one that matters:

- **covered**: the audit ran and findings are recorded, including "came back clean". Clean is a result, not an absence of work.
- **not applicable**: triaged out at intake, **with the reason**. Tier and domain decide this, not time.
- **not done**: intended but not completed. Ran out of scope, blocked, or deprioritised.

### Publish it

The coverage table goes in the report's method section, in full, including every `not done` row.

This is the part that makes it work. A gap the assessor knows about is a discipline problem. A gap the reader can see is a stated limit, and the reader can decide whether it matters. It also removes the temptation to quietly drop a dimension that was hard, because dropping it now requires writing the word "not done" where the client will read it.

A report claiming nineteen dimensions with no coverage table is asserting completeness it has not demonstrated, which is the same failure as an unverified number.

### Check it twice

Once **after the fan-out and before writing**, when a gap is still cheap to fill. Once **at publication**, as part of the gate. The mid-point check is the useful one; the final check usually only confirms what the first one found.

## Check the precondition before applying a rule

Every rule of thumb has assumptions. Applying one whose precondition fails hands a competent defender an easy rebuttal and discredits the surrounding findings. Check first.

| Rule of thumb | Precondition to verify first |
|---|---|
| Never use float for money | Is the SQL type `DOUBLE` or `FLOAT`? Are values whole units? Does the code round where fractions arise? Compare magnitude against 2^53 and 2^24. Integers are exact in a `DOUBLE` well past any realistic revenue figure. |
| Missing foreign keys break integrity | Were they ever present, or never added? Raw SQL joins work without them. Establish what actually degrades: enforcement, ORM traversal, or indexing. |
| Type checking is disabled so nothing is checked | Search for a compensating gate: release script, pre-commit hook, CI workflow, a separate typecheck task. One may exist and block correctly. |
| No tests means nothing is verified | Look for executable specifications under other names: check scripts, assertion harnesses, scenario runners. They may exist and be unwired. |
| The feature is missing | Is the field or setting **read** anywhere, or only written? Absence of a read is the finding; absence of a screen is not. |
| A comment says it was done | An instruction is not execution evidence. Establish separately, and state the test that would settle it. |
| A dependency from a URL is a supply-chain risk | Does the lockfile carry an integrity hash? Is it the vendor's official distribution channel? The real risk is usually scanner blind spots, not tampering. |
| Too many tables means over-normalisation | Measure structural duplication by pairwise field overlap. Wide and redundant are different conditions. |
| Swallowed errors mean symptoms were suppressed | Date them. A swallow written with the code it wraps is a defensive habit; one added later in a crash-fixing commit is suppression. The ratio is the finding. |
| Constraints were removed under pressure | Count removal events in history. Zero drops refutes it outright. Falling density over time is a different and softer finding. |
| There are no supply-chain defences | Look for them before reporting absence: release-age cooldown, install-script allowlist, `ignore-scripts`, overrides, frozen installs, private registry, secret scanning. Then check each against the lockfile, since a control can be present and no longer aligned with what is installed. |

When a precondition fails, the finding usually survives in a changed form and often a stronger one. Restate it, do not delete it.

## Second method for decisive findings

Any finding that changes the verdict must be confirmed by a **second independent method**, not a second look at the same evidence.

Examples that satisfy this:

- Reading the code, then running the arithmetic to reproduce the defect numerically.
- Counting from the schema, then counting from the generated migration SQL.
- Grepping for a pattern, then reading the git history of the file that contains it.

Examples that do not: re-reading the same file, or asking a second subagent the same question.

## Adversarial check before publishing

For each headline finding, write down the strongest rebuttal a competent, motivated defender would offer. If you cannot answer it from evidence already gathered, the finding is not ready.

Rebuttals that have landed in practice:

- "Foreign keys do not prevent joins." Correct, and fatal to the claim as worded.
- "Nobody ever removed a constraint." Correct, and refutes the removal theory.
- "We run the type check before every release." Correct, and there is a script proving it.
- "The table count is high because the product is wide." Correct, if duplication was never measured.

## Pre-publication gate

Run this before any deliverable ships, and again after every correction.

1. The coverage ledger is complete, every dimension has a status, and it is reproduced in the report.
2. Every number in the document appears in the ledger at status `verified`.
3. Every `unverifiable` claim carries its limit inline and names the settling test.
4. Every rule of thumb applied has its precondition recorded as checked.
5. Every decisive finding has two independent confirmations.
6. Every headline has a written rebuttal you can answer.
7. A strengths section exists and is specific.
8. A withdrawn-claims section exists, even if empty.
9. Grep the full deliverable set for any superseded figure or phrasing.

Step 9 matters more than it looks. Assessments run long, conclusions change, and a corrected finding left standing in a second document is an inconsistency a defender will find.

## When to withdraw

Withdraw rather than soften when any of these hold:

- The precondition for the underlying rule does not hold in this system.
- The evidence is an instruction, an intention or a comment rather than an observation.
- The claim rests on a subagent figure you cannot reproduce.
- A competent defender's rebuttal is correct.

Record every withdrawal and publish the list. It is the strongest credibility signal an assessment carries, and it pre-empts the accusation that the work was directional rather than evidential.
