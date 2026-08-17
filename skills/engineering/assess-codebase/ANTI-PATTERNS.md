# Anti-patterns

Failure modes of the assessor, not of the codebase. Every one below was committed in a real assessment and caught only because someone pushed back.

## Trusting a number you did not verify

The single most common source of error. In one assessment, five published figures were wrong, and **every one came from a subagent report rather than from a check run personally.** Every figure verified directly held up.

Subagents are fast and thorough, and wrong often enough to matter. They miscount, they scope a grep differently than they describe, and they round a conclusion toward the hypothesis they were given.

Before any number reaches the output, run the measurement yourself. If that is impractical for a figure, either drop it or mark it as unverified in the text. An unverified number in a report is worse than a missing one, because it is the thing a defender will check first.

## Applying a general rule without checking its preconditions

"Never use float for money" is sound guidance with assumptions underneath it. In one system the money columns were `DOUBLE` rather than `FLOAT`, the values were whole units, and the code rounded at the only calculation that produced fractions. Integers are exact in a `DOUBLE` to 2^53, so there was no precision error at all. The finding was published as a correctness defect and had to be downgraded to a consistency one.

Every rule of thumb encodes a precondition. Check it. The version that survives contact with a competent engineer is worth ten that do not.

## Reading intent into a comment

A file saying "run this in phpMyAdmin against the LIVE database" is an **instruction**. It is not evidence that anybody did.

The same error in reverse: treating an explanatory comment as proof that a decision was deliberate and considered. A comment explaining a choice tells you someone wrote a sentence, not that anyone weighed the alternatives.

State what the artefact is, then state separately what it would take to establish execution. Frequently a single query settles it, and offering that query is worth more than the accusation.

## Asserting a cause instead of testing it

A plausible story about why a system is the way it is will be believed, including by you. In one assessment the working theory was that constraints had been removed to keep the interface working. Four passes over the commit history refuted it: zero constraints were ever dropped, and the permissiveness was present in the first commit.

The theory was right about the outcome and wrong about the mechanism, and the mechanism is what determines the fix. Test causal claims against history before publishing them, and report the refutation as prominently as a confirmation.

## Answering the example instead of the class

When someone says "there is no Button component", they are handing you one specimen of a category. Auditing buttons and reporting back on buttons misses the finding.

In one assessment, generalising two named examples produced both the most severe security finding in the review and an entire category nobody had considered. Neither would have surfaced from answering the questions as asked.

For every concern raised: name the category, audit the category.

## Leading with a claim a defender can rebut

Two examples. "No foreign keys means we cannot do joins" is false; raw SQL joins work fine without them. "There are too many tables" is not supportable when the schema shows almost no structural duplication.

Both sat next to real findings, and both would have cost the room. The accurate versions were stronger: the ORM could not express the relationships, so they were reimplemented by hand at hundreds of sites and each one fails silently; and the schema is wide because the product is wide.

Sanity-check every headline against a competent, motivated defender before it goes in.

## Reporting only problems

A report with no strengths reads as advocacy and gets discounted accordingly. It also fails at the question that actually matters, which is repair or replace.

The strengths section is not a courtesy. In one assessment it was the reason the recommendation was "salvage" rather than "rewrite", and that distinction was worth more than any individual finding.

## Softening instead of withdrawing

When a finding cannot be substantiated, hedging it is worse than removing it. Hedged findings survive into summaries with the hedge stripped off.

Withdraw it and say so. A "claims withdrawn during review" section is the strongest credibility signal an assessment can carry, and it pre-empts the rebuttal that the work was directional rather than evidential.

## Confusing severity with alarm

Rank by how silently something can be wrong. A crash is loud and gets fixed within the hour. A forecast inflated by a large constant factor still looks like a number, gets acted on, and can survive for months.

A control that reports success it did not achieve outranks both, because it consumes the attention that would otherwise find the problem.

## Writing for the author

The audience is usually the person who must decide, often not an engineer and frequently not the author. Findings need to survive being read by someone who cannot evaluate the code.

That means the verdict goes first, the two or three findings that decide it are quoted from source, and each argument carries a note on how hard it is worth pushing. Stakeholders need to know which points to spend credibility on.

If a plain-language layer is wanted, keep the substance in the technical prose and let the plain layer carry only the analogy. Otherwise removing it later strips out facts.

## Letting the deliverable drift from the findings

Assessments run long and conclusions change. Every correction has to propagate to every document that carries the claim, or the set becomes internally inconsistent and each inconsistency is a rebuttal.

After any correction, grep the whole deliverable set for the old figure and the old phrasing. Republish everything affected, not just the document you were looking at.

## Searching only the formats you find comfortable

Requirements live where the author was comfortable writing them. For a non-technical author that is Word, Excel or PowerPoint, not markdown.

In one assessment a repository held two dozen Word documents, a feature spreadsheet and a slide deck. A markdown-only reading returned none of them. The technical content happened to be mirrored in markdown, so nothing critical was lost, but that was luck rather than method, and the spreadsheet carried the business view, what each module was for and what problem it solved, which is exactly the material intake and the executive summary need.

Office formats are ZIP archives of XML and read with the standard library alone. There is no excuse for skipping them.

## Sanitising by pattern instead of by class

Before publishing anything derived from a real engagement, the client-identifying detail has to come out. The failure is to remove the identifiers you happened to think of, then declare it clean.

In practice: a first pass removed company names, code identifiers and four specific figures, and a scripted sweep reported clean against exactly those patterns. A reader then found a reimplemented-relationship count, an inflation factor, a dependency-tree size, a component ratio, and a translation table still written as findings from a real system rather than as illustrations. None of them matched the patterns the sweep tested.

This is the same error the rest of this file warns about, turned on the assessor's own output: checking the instances you named rather than the category they belong to.

Sanitise by class, not by instance:

- **Every quantity**, not the memorable ones. Strip code fences and look at every number of three digits or more, every ratio, every "N times" and every written-out multiplier.
- **Every example framed as something that happened.** An illustration says "a calculation returning a constant where it should vary"; a disclosure says "their board pack overstated the forecast fortyfold". Both teach the same lesson and only one names a system.
- **Every domain noun.** The vocabulary of the client's industry is as identifying as its name.
- **Then have someone else read it.** The author knows what was removed and reads past what was not. This one was caught by a reader, not by a script.

## Counting things instead of running them

Structural probes are seductive because they scale: a grep produces a ratio, a ratio looks like evidence, and a page of ratios reads as thorough. They are also blind to an entire class of defect.

A formula that computes the wrong thing has no structural signature. It passes type checking, it has no unusual shape, and no count will surface it. It can only be found by reading the calculation and executing it with realistic inputs.

In one test of this method, a run that covered every dimension and produced a substantial, well-evidenced findings list missed a calculation that had been returning a fixed value on every day of every year since the system was written, inflating a headline figure by more than an order of magnitude on the screens most senior people read. Everything structural around it was found. Nobody ran the arithmetic.

The correction is not more greps. It is one hour, spent deliberately, listing the calculations whose output people act on and running each of them. A clamp firing on every ordinary input, a magic constant in date arithmetic, and a widely-called function with one commit are the three tells worth knowing.

The same blindness applies to compounding. Six findings listed separately understate the risk of all six landing on the same routine operation. Trace one operation end to end and count the layers that would catch a failure; zero or one is a different statement from six bullet points.

## Delivering findings instead of an assessment

A list of findings answers what is wrong. It does not answer why it will happen again, or what the person inheriting the system actually faces. Those are separate questions, and neither falls out of the audit work by itself.

The failure is subtle because nothing is missing. In one test of this method the run measured the constraint decline, the reading load of the schema, the untouched modules and the self-contradicting documentation, and reported all four correctly, each inside a different finding. Nobody assembled them. The report told the reader that constraint density fell, and never told them why, or that a team of three would meet the same limit, or that the tool which built the system cannot safely maintain it either.

The measurements were all there. The two conclusions that a technical leader actually plans from were not.

Make synthesis an explicit step before drafting, and treat both sections as required output rather than as commentary if there is room. The causal section is also what stops the report reading as a judgement on a person, which is frequently what decides whether any of it gets acted on.

## Recommending repair or replace, as though those were the options

The disposition is usually presented as a binary, and the binary is wrong often enough that it is worth treating as a default error.

The missing option is absorbing the proven functionality into a platform the organisation already runs. It is missed for a simple reason: the assessor is looking at one repository and never asks what else exists. Nothing in a codebase tells you that another team already operates a platform with continuous integration, tests, deployment, monitoring, a data layer and recorded decisions, which is precisely the list the findings say this system lacks.

Two things make it the right answer more often than it is chosen.

Same language is not the same platform. Two systems can share a language and a UI library and still differ in routing, data loading, build, deployment, conventions and half their dependencies. Recommending repair commits the organisation to operating a second platform permanently, and that ongoing cost never appears in the repair estimate.

Absorb is frequently faster than repair, which inverts the usual assumption. The expensive part of building software is working out what the business actually needs, and a system people have used has already paid it. Porting from working software is transcription against a reference, not construction from a specification, while the receiving platform supplies the foundations the findings list as missing.

The correction is an intake question, not a technique: ask what else the organisation runs before writing any recommendation. Then produce the split the choice requires, commodity against differentiating, since the commodity half usually should not move at all.

A related failure in the executive summary: giving five reasons because five findings felt important. Three is what survives a meeting. Findings that cost the business the same thing are one reason regardless of their technical cause, and a reason still naming a mechanism, backups, transactions, tests, has not finished being translated into money and time.

## Judging technology in the abstract

An assessment that reports a database engine, a hosting model or a runtime without reference to what the organisation already runs has produced an inventory, not a finding.

The technologies in these systems are usually fine. They were chosen by a model optimising for something that works, and what works is generally mainstream. Reporting them as neutral facts, or worse defending them as reasonable, misses the actual cost: adopting a choice that differs from an established standard means operating two of everything, permanently, and that tax never appears in a repair estimate.

Worse is the case where the organisation has already migrated away from the choice. Then adoption does not merely add a second thing to operate, it reverses work that has been completed and paid for, and reopens an argument that was settled. Divergence and regression should be labelled differently because they cost differently.

The correction is an intake question rather than a technique: ask what the standard is and what is deliberately being exited, before forming any view on the technology. Then ask whether the choice is a configuration or is architected in, because that distinction, not the technology, sets the cost of changing it.

## Mistaking import machinery for a migration plan

Where existing data has to be brought in, the migration is frequently the largest single item in the programme and the last one examined.

The trap is that the scaffolding looks like readiness. Staging tables, batch tracking with accepted and rejected counts, mapping and alias tables, a handful of source-identifier columns: all present, all sensible, and none of it evidence that anybody opened the source system.

The distinction is between machinery and specification. Generic staging that accepts arbitrary input and maps it afterwards is precisely what gets built when the shape of the source is unknown. It is the flexible choice, and its flexibility is the tell.

In one assessment the finding was reported as an absence of source identifiers, which turned out to be wrong on the count and to be answering a smaller question than the one that mattered. The identifiers existed. What did not exist was any evidence that the incumbent system's schema had ever been looked at, which is what decides whether the migration is a transformation exercise or a discovery exercise first.

Ask for the source schema and a row count at intake. Then check for granularity mismatches, identifier strategy and whether both systems will run in parallel, because each of those is a redesign rather than a mapping and none of them is visible from the target codebase alone.

## Crediting a test plan as testing

Test checklists, UAT packs and acceptance criteria are inexpensive to produce and frequently exist in quantity. Their presence proves that somebody anticipated testing. It proves nothing about whether testing happened.

In one assessment several comprehensive checklists covered workflows, permissions, acceptance and smoke coverage. Every one had been written on the day the system was first committed, none had been touched since, and across all of them not a single item was ticked. The plans were thorough. The execution was zero, and an assessment that listed the checklists under documentation would have credited the system with verification it never had.

Check for execution rather than intent: ticked against unticked, the date the file was last touched, whether there are result, tester and date columns rather than only steps, and whether a real pass left the artefacts it leaves behind, screenshots, a defect log, issues raised while testing.

The finding is worth reporting because it cannot be argued with. Zero ticked out of several hundred needs no interpretation, and the consequence follows directly: nothing has confirmed that what the screens display matches what the data holds, so every figure a user has seen is unverified.
