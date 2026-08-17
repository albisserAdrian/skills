---
name: assess-codebase
description: Whole-system assessment of an existing codebase for production fitness. Starts by establishing what the business does, what the system is for and who supports it, then audits security, data integrity, correctness, platform, practice and governance against the norms for that class of system. Produces a non-technical executive summary alongside the technical report. Use when asked whether an application is production ready, to review an entire app rather than a change, to assess something inherited or AI-generated, or to produce a go/no-go for management. For reviewing a single change or diff, use `review` instead.
---

# Assess codebase

**No figure reaches the output unless you ran the measurement yourself and can restate the command. Subagents find things; they never supply numbers. A refuted hypothesis is worth as much as a confirmed one, and an honest withdrawal buys credibility for everything else.**

This is not a bigger code review. A code review asks whether a change improves the codebase. This asks whether a whole system can be trusted to run a business, which is a different question with different evidence and a much higher cost for being wrong.

**Read `SAFE-CONDUCT.md`, `INTAKE.md` and `VERIFICATION.md` before starting.** Safe conduct first: an assessment reads an entire codebase and runs commands inside it, so the assessor is a risk to the client. Never install the target's dependencies, never connect to production, never print a secret. Intake sets what the system is and therefore what the bar is. Verification holds the rules that keep the output accurate, and is not optional. In the assessment this skill was built from, every published error traced to a number accepted from a subagent without independent measurement. Every figure measured directly held up.

## When this applies

- "Is this ready for production?"
- "Review the whole app, not just this change."
- "We inherited this. What did we inherit?"
- "This was built fast with AI. Is it safe?"
- Producing a go/no-go a non-engineer will read and act on.

If the target is a diff, a branch or a pull request, use `review`. If the target is security only, `security` has the OWASP patterns; this skill covers security as one dimension of many.

## The starting assumption: nothing was planned

With a conventionally built codebase you can assume a baseline and audit the exceptions. Someone chose the patterns, someone reviewed them, and the deviations are what need attention.

**Assume no baseline exists.** Every practice is a hypothesis to be tested: transactions, indexes, pagination, validation, error surfacing, authorisation on each path, consistent conventions. Each is either demonstrably present or absent, and only a check distinguishes them. This holds whoever built it, and it is mandatory for anything AI-generated or built without engineering review.

Two rules follow.

**Never infer a practice from one instance.** Consistency is the property in question. Finding a transaction proves a transaction exists, not that multi-step writes are transactional. Sample across modules and report a ratio.

**Never infer intent from a comment.** A comment describing a design decision proves a sentence was written.

The reason this matters is structural rather than a judgement about care. When a system is directed by someone specifying user outcomes, the brief covers what the screens do and never covers what happens at fifty concurrent users, at ten million rows, when an import dies halfway, or when somebody who did not build it has to find a wrong figure. **The functional requirements are usually met; the non-functional ones were never articulated.** That is why the interface can be genuinely good while the system underneath cannot be operated, and it tells you exactly where to look. See `NON-FUNCTIONAL.md`.

It compounds, too. Those properties are all global: a transaction spans several writes, convention consistency spans modules, a shared component must be known about to be used. They are handled early while the system fits in view and skipped later as it does not. **Sample the newest modules specifically**, because they are simultaneously the least reviewed, the least exercised and the least likely to carry these properties.

## The method

**Fan out, then verify.** Assessment work parallelises well and verification does not. Run many narrow audits at once, then personally check every number that will appear in the output.

0. **Intake, before opening the code.** Read the repository's own account first: README, `docs/`, any roadmap or feature list. Build a claimed-feature inventory, note any document that contradicts itself, and enumerate the functional domains, because reliability means something different in records, operations, capture, evidence and reporting. Then ten short questions establishing what the business does, what the system is for, whether it is the system of record, what flows through it, who uses it, who supports it, and what is driving the timeline. From the answers, set a criticality tier and establish the norms for this class of system. See `INTAKE.md`. Skipping this produces a report calibrated to nothing: the same defect is a blocker in a booking ledger and a backlog item in a comment log, and you cannot tell which without asking.
1. **Establish scale first,** measuring it yourself. Lines, files, tables, routes, dependencies, commit count and date range. Everything after is calibrated against this, and the numbers themselves are often the finding.
2. **Dispatch parallel audits, one dimension each.** See the dimension list below. Give each a single question, the facts already established so it does not re-derive them, an instruction to report what came back clean, and an explicit invitation to refute the premise. State in every brief that all quantities will be independently re-measured, so the agent optimises for locating evidence rather than for producing figures.
3. **Track coverage as you go, not from memory at the end.** One row per dimension, status `covered` (including clean), `not applicable` with the reason, or `not done`. Check it once after the fan-out while a gap is still cheap to fill, and again at publication. See `VERIFICATION.md`.
4. **Re-measure everything before it is written down.** Subagent output is a set of leads, not results. Maintain the claim ledger in `VERIFICATION.md`; nothing enters a deliverable at status `claimed`.
5. **Check the precondition before applying any rule of thumb.** The precondition table in `VERIFICATION.md` covers the rules that have failed in practice. A finding whose premise does not hold discredits the ones that do.
6. **Follow the surprises.** The most serious finding in an assessment is usually not on anyone's checklist. It surfaces when a verification step contradicts an assumption, which is another reason to re-measure rather than accept.
7. **Confirm decisive findings by a second independent method.** Reading the code and then reproducing the defect numerically counts. Reading the same file twice does not.
8. **Calibrate severity, including downward.** Withdrawing a finding you cannot substantiate makes the rest defensible.
9. **Run the pre-publication gate,** and run it again after every correction. Corrections that fail to propagate across the deliverable set become rebuttals.
10. **Write for the person who will act.** Usually not the author, often not an engineer.

## Dimensions

Cover all of these. Each is one audit, one question.

| Dimension | The single question |
|---|---|
| Security | What can an attacker or a curious employee reach? |
| Data integrity | Can the data be trusted after months of multi-user use? |
| Correctness | Can the numbers this system reports be relied on? |
| Never-executed surface | How much of this has actually run? |
| Operational readiness | At 2am, can a failure be detected, diagnosed and recovered? |
| Performance | Does it survive real volumes on real hardware? |
| Platform | Can it be deployed, scaled, and restored? |
| Schema and types | Do the column types suit the domain? |
| Maintainability | What does a safe change cost, and is that trend rising? |
| Shared abstractions | What should be shared and is not? |
| Frontend | Is there a component system, or is every screen hand-built? |
| Migrations | What does the schema history reveal about data-loss risk? |
| Supply chain | What executes at install time, which defences exist, and are they still aligned? |
| Secrets handling | Where do credentials live, and what does rotating one cost? |
| Governance | PII, retention, erasure, audit integrity, separation of duties |
| Onboarding | Could a new team take ownership, and from where? |
| Concurrency | What happens when two people act at once? |
| Diagnosability | A figure is wrong and the author is unavailable. Where does someone start? |
| Claimed against actual | For each feature the project says it built, does the mechanism exist? |
| In-flight work | What is lost when the process is replaced mid-action, and what is the user told? |

Nineteen is the full set, not a mandatory one. Triage by tier and by the domains found at intake.

**Always:** security, secrets handling, claimed against actual, diagnosability, operational readiness.
**Tier 1, or any domain handling money, evidence or irreplaceable data:** add data integrity, correctness, concurrency, in-flight work, migrations, governance.
**Add when the intake answer warrants it:** performance and platform where growth or scale is expected; frontend and shared abstractions where cost of ownership is the question; supply chain and onboarding where a team is inheriting it.

For a listed company or anything touching financial reporting, governance is not optional and the bar is IT general controls, not code quality.

## Five disciplines that decide the quality of the output

### 1. Review classes, not examples

When a stakeholder names a concern, they are giving you a specimen, not the specification. "There is no Button component" means *audit every shared abstraction*. "Can it run on Fargate" means *audit the whole platform story*: environments, infrastructure as code, secrets, network exposure, recovery objectives, vendor lock-in.

Answering only the named example is the most common way an assessment misses its worst finding. Ask of every concern raised: what is the category this belongs to, and have I covered the category?

### 2. Ask whether each control does what it reports

The most damaging findings are not broken features. They are working screens in front of absent mechanisms: a backup that records success without writing bytes, an encryption toggle whose field no code reads, single sign-on with admin pages and no authentication library.

The probe is mechanical. For every setting, status or control surface, find where the value is **read**, not where it is written. A field that is only ever written is decorative.

These matter disproportionately because they are exactly the artefacts offered as evidence that a control exists.

### 3. Ask whether the code has ever run

Static quality says nothing about whether a path has been exercised. In fast-built systems, whole modules are generated, wired into navigation and never opened.

Git iteration rate is the cheapest proxy. Code that runs gets fixed. A module with one commit in a project with a thousand either worked perfectly first time or was never executed. Cross-check against tables never written to, stub markers, and functions with no callers.

### 4. Use history as evidence, not decoration

Do not assert why a system is the way it is. Test it. `git log -S` dates when a pattern entered. Comparing a measure at the first commit against HEAD distinguishes "never had it" from "lost it", and those have completely different remediation implications.

Bucket a metric across thirds of the history to find trends. A constraint density that falls over time is a finding; a constant one is a starting condition.

### 5. Separate outcome from mechanism

A stakeholder's theory about *why* is often right about the end state and wrong about how it got there. Both halves matter: the outcome sets the risk, the mechanism sets the fix. "Constraints were removed" and "constraints were never added" produce the same system and completely different remediation plans, because one has a prior state to restore and the other does not.

## Calibrating severity

Rank by **how silently something can be wrong**, then scale by the criticality tier set at intake.

- A wrong number that looks plausible outranks a crash.
- A missing control outranks a weak one.
- A control that reports success it did not achieve outranks both.
- Anything invisible to the people operating the system gets a severity bump.

Downgrade honestly. Before applying a general rule ("never use float for money", "always parameterise"), check whether its preconditions hold here. Rules of thumb have assumptions, and a finding whose premise fails hands the other side an easy rebuttal. The precondition table is in `VERIFICATION.md`.

The finding does not change with the tier; its severity does. No automated backups is a blocker for a system of record and a should-fix for a comment log. Applying the system-of-record bar everywhere produces a report that reads as alarmist, and the real findings get discounted along with the inflated ones. See the tier table in `INTAKE.md`.

Weigh the support model separately. Every finding that depends on a person noticing or acting is effectively unmitigated when there is nobody to notice. That alone can turn a pass into a fail, independently of anything in the code.

### The scale

Use one vocabulary and define it in the report. Severity is silence multiplied by consequence at this tier, not how alarming the finding sounds.

| Level | Meaning |
|---|---|
| **Blocker** | The system cannot be trusted for its tier. Go-live stops until it is resolved. |
| **Serious** | Real exposure. Fix before scale, before the next period close, or before the data becomes irrecoverable. |
| **Should fix** | Genuine cost or risk. Schedule it. |
| **Backlog** | True, and low consequence at this tier. |
| **Observation** | Recorded because it is worth knowing. No action implied. |

Resist promoting everything. A register where most items are Blocker is a register nobody can plan from.

## What a pass looks like

**Every example in this skill comes from one system that failed.** That is a real bias, and left uncorrected it produces false positives, which is dishonest in the opposite direction to missing findings. Guard against it deliberately.

The examples here illustrate *probes*, not predictions. A codebase not matching them is the expected result for competent work, and should be reported as such rather than hunted around.

### Run the probes in both directions

Every measurement that detects absence also detects presence. Record both, from the same command, at the same time. This is the correction for finding problems systematically and strengths anecdotally.

| Probe | Absence | Presence, equally worth reporting |
|---|---|---|
| Relations against link columns | Integrity unenforced | Referential integrity is enforced and can be relied on |
| Multi-step writes inside a transaction | Partial-failure risk | Writes are atomic |
| Shared component adoption ratio | Duplication and cost | A design system exists and is genuinely used |
| Where a setting is read | The control is decorative | The control does what it reports |
| Suppressed errors | Failures are invisible | Failures surface and are actionable |
| Module iteration rate | Never exercised | Actively used and maintained |
| Claimed features against mechanisms | Claimed but not built | The documentation is accurate, which is rare and load-bearing |

### Positive indicators to look for actively

These take minutes and materially change the recommendation:

- A test suite covering business rules, not only utilities.
- Continuous integration that blocks on failure, with a history showing it has blocked.
- Consistent conventions across modules built months apart, which indicates something is holding the standard.
- Documentation whose claims survive checking.
- A reconciliation job comparing derived figures against source.
- Errors reaching an error-tracking service.
- Constraint density flat or rising across the history rather than falling.

Any three together usually mean the system is in better shape than a findings list implies, and the report should say so in the first screen rather than in a strengths section at the end.

### Reporting a pass

If the evidence supports it, say so plainly and briefly. "Fit for purpose at this tier. Three should-fix items, none blocking. Recommend proceeding." A format that can only express alarm is not an assessment method, it is a template for one outcome.

For a Tier 3 informational system, expect most correctness and recovery findings to land at Backlog. That is the correct result, not a failure to look hard enough.

### When to stop

Stop when the remaining dimensions would not change the recommendation. Scale depth to the tier and to what is at stake: a Tier 1 system of record justifies the full set, a Tier 3 tool does not.

If three dimensions in a row come back clean and nothing so far reaches Serious, say so and finish early. Continuing past that point produces padding, and padding is what gets a report discounted.

## Output

Three deliverables, because three audiences.

**An executive summary.** One page for people who will never read the technical report. They are not deciding whether the code is good; they are deciding whether the business loses money, breaks a commitment or fails an audit. At most five reasons, each stated as a consequence rather than a condition, with cost and likelihood. No file names, no counts, no framework names. See `EXECUTIVE-SUMMARY.md` for the translation table and a worked example.

**A decision document.** Verdict in the first screen. The two or three findings that actually decide it, each verified and quoted from source. Conditions under which a limited rollout would be defensible. A calibrated position table saying how hard to hold each argument, because stakeholders need to know which points are worth spending credibility on.

**A remediation register.** Every finding, grouped by the team that would own it, with effort and sequencing. Sequencing matters more than severity: visibility and backups come before repairs, because until failures are observable no fix can be confirmed.

### Stating effort honestly

**Give effort two ways: with AI assistance and without.** The multiplier is not uniform, and quoting a single figure hides which parts are actually cheap.

AI assistance compresses the mechanical and repetitive work heavily: migrating a thousand call sites onto an existing component, adding indexes across a schema, writing tests that characterise behaviour already in the code, converting column types. It compresses the judgement work barely at all: deciding the money representation, designing the referential integrity strategy, determining which of three revenue lineages is authoritative, working out what a figure was supposed to mean.

That asymmetry is not incidental. The judgement work is missing precisely because it requires holding the whole system in view, which is the thing that failed the first time and will not be fixed by applying the same tool faster.

**Separate effort from calendar.** Effort assumes people exist. Hiring engineers who can work safely on an undocumented system of this size takes months, and their first weeks are ramp-up. State the calendar with hiring lead time included, and say explicitly who does the work in the interim, because that gap is usually where a plan quietly fails.

### Recommend one dynamic test

Everything in a static assessment is inference. For any system of consequence, recommend a **load test against production-shaped data volumes** before go-live: a copy of real data, realistic concurrency, the screens people actually use.

It settles in a day what static analysis can only argue: whether the connection pool exhausts, which queries fall over at real row counts, whether imports survive real volumes, and what the failure actually looks like when it comes. It also converts the arithmetic findings, such as a page that issues a thousand queries, into an observed fact that nobody can dispute.

### Address the human transition

A recommendation that ignores the people involved does not get followed.

Somebody built this, is usually proud of it, and is frequently the product owner and the only holder of the domain knowledge. People are already using it. A report that reads as a verdict on a person rather than on a system will be resisted regardless of its evidence.

State plainly what happens to the users during the transition, and frame the builder's role going forward as the knowledge asset it genuinely is. The technical recommendation is often correct and rejected on these grounds alone.

All three must contain:

- **What is genuinely good.** This determines whether the answer is repair or replacement, and a report with no strengths reads as advocacy and gets dismissed.
- **The coverage table.** Every dimension with its status, including every `not done`. A report asserting completeness without showing coverage is claiming something it has not demonstrated.
- **Method and limits.** What was static-only, what could not be run.
- **Claims withdrawn.** List conclusions revised during the work. This is the single strongest credibility signal in an assessment, and it pre-empts the rebuttal.

## Honesty

- **Verify or drop it.** An unverified number in a report is worse than a missing one, because it is the first thing a defender checks.
- **Never hedge a claim you cannot substantiate.** Hedged findings survive into summaries with the hedge stripped off. Withdraw instead, and publish the withdrawal.
- **No implied intent.** A comment saying "run this in production" is an instruction, not evidence anybody did.
- **Do not lead with a rebuttable claim.** Check that each headline survives a competent defender. "No foreign keys means no joins" is false and costs you the room.
- **Report the refutations.** If the evidence contradicts the brief you were given, say so plainly and early.
- **Name the limits of static analysis.** Several findings usually resolve in minutes against a running system; say which.
- **Do no harm gathering evidence.** A finding is not worth executing an untrusted dependency tree, touching production, or putting a live credential into a transcript.

## Reference files

| File | What it holds |
|------|---------------|
| `SAFE-CONDUCT.md` | **Read first.** Rules for not harming the client: no installs, no production, no printed secrets, toolchain hygiene, report distribution |
| `INTAKE.md` | **Read second.** The ten questions, criticality tiers, support-model weighting, establishing domain norms |
| `VERIFICATION.md` | **Read third.** The verification rule, subagent contract, claim ledger, precondition table, publication gate |
| `NON-FUNCTIONAL.md` | The six stresses the original brief never covered: concurrency, in-flight work, time, load, real data, diagnosability |
| `EXECUTIVE-SUMMARY.md` | The non-technical one-pager: format, technical-to-consequence translation, worked example |
| `PROBES.md` | The concrete measurements and commands, by dimension |
| `ANTI-PATTERNS.md` | Failure modes of the assessor, drawn from real mistakes |
