# Anti-patterns

Failure modes of the assessor, not of the codebase. Every one below was committed in a real assessment and caught only because someone pushed back.

## Trusting a number you did not verify

The single most common source of error. In one assessment, five published figures were wrong, and **every one came from a subagent report rather than from a check run personally.** Every figure verified directly held up.

Subagents are fast, thorough and confidently wrong at a meaningful rate. They miscount, they scope a grep differently than they describe, and they round a conclusion toward the hypothesis they were given.

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

In one assessment a repository held twenty-five Word volumes, a feature-inventory spreadsheet and an executive deck. A markdown-only reading returned none of them. The technical content happened to be mirrored in markdown, so nothing critical was lost, but that was luck rather than method, and the spreadsheet carried the business view, what each module was for and what problem it solved, which is exactly the material intake and the executive summary need.

Office formats are ZIP archives of XML and read with the standard library alone. There is no excuse for skipping them.

## Sanitising by pattern instead of by class

Before publishing anything derived from a real engagement, the client-identifying detail has to come out. The failure is to remove the identifiers you happened to think of, then declare it clean.

In practice: a first pass removed company names, code identifiers and four specific figures, and a scripted sweep reported clean against exactly those patterns. A reader then found a reimplemented-relationship count, an inflation factor, a dependency-tree size, a component ratio, and a translation table still written as findings from a real system rather than as illustrations. None of them matched the patterns the sweep tested.

This is the same error the rest of this file warns about, turned on the assessor's own output: checking the instances you named rather than the category they belong to.

Sanitise by class, not by instance:

- **Every quantity**, not the memorable ones. Strip code fences and look at every number of three digits or more, every ratio, every "N times" and every written-out multiplier.
- **Every example framed as something that happened.** An illustration says "a calculation returning a constant"; a disclosure says "the forecast was overstated twentyfold". Both teach the same lesson and only one names a system.
- **Every domain noun.** The vocabulary of the client's industry is as identifying as its name.
- **Then have someone else read it.** The author knows what was removed and reads past what was not. This one was caught by a reader, not by a script.
