# Intake

Do this before opening the code. The same defect is a blocker in one system and a backlog item in another, and you cannot tell which without knowing what the system does, who runs it, and who fixes it when it breaks.

Keep it short. Eight questions, a few minutes. Where the answer is unknown, record the assumption you proceed under and put it in the report's limits section.

## Ask these

1. **What does the business do, and what is its regulatory footing?** Sector, rough size, listed or private, any regime that applies (privacy, financial reporting, payment data, health).
2. **What will this system be used for, specifically?** Not the product name. The job it does.
3. **Is it the system of record, or does something else hold the truth?** If a figure here and a figure elsewhere disagree, which one wins, and who decides?
4. **What flows through it?** Money, inventory, customer obligations, staff data, or only commentary and notes. Ask for the largest single number the system will hold or influence.
5. **Who uses it?** Roles, rough headcount, internal only or customer-facing. Customer-facing changes the security bar sharply.
6. **What does it replace?** A previous system, spreadsheets, or nothing. What happens operationally if it is unavailable for a day.
7. **Who supports it?** In-house engineers and how many, an agency, one person, or nobody. Who is called at 2am. This changes the recommendation more than most technical findings.
8. **What is driving the timeline, and what happens if it slips?** Distinguishes a real constraint from an assumed one.

If the requester cannot answer 3, 4 or 7, that is itself a finding worth stating.

## Then ask about data and integrations

Two questions the code cannot answer, and both change the remediation materially.

9. **What data has to come in, from where, and in what state?** Years of history from a legacy platform is a different exercise from starting empty. Ask what the mapping is, how it will be validated, and whether it is reversible. Then check the schema for the thing that makes reconciliation possible at all: a source-system identifier on core records. Its absence is a blocker for any migration, and it is invisible until somebody tries.

10. **What must this system talk to, in which direction?** Finance, billing, the ad server, the traffic system, identity, email. Build the map from the business answer, then compare it against what exists in the code. A gap in either direction is a finding: an integration the business assumes exists and does not, or one built against a system nobody intends to keep.

## Product ownership

In systems built this way the person who directed the build is usually also the product owner, and frequently the only person who knows why a rule exists. Establish whether that holds.

If it does, treat it as an asset rather than a risk in the first instance. It means the domain knowledge is available, which is the difference between a team that can take ownership and one that has to reverse-engineer intent from code. Capture it deliberately and early.

It also means nothing independent has been checking scope, so expect features that were built and never used. Confirm against the never-executed findings before recommending anyone spend money making them work.

## Read what the repository claims about itself

Before any audit, read the repository's own account: `README`, everything in `docs/`, any roadmap, feature list, changelog, architecture note, and agent-context file such as `CLAUDE.md` or `CONTEXT.md`.

**Search for Word, Excel, PowerPoint and PDF, not only markdown.** Non-technical authors write requirements where they are comfortable writing, and that is rarely a `.md` file. A repository can hold twenty-five Word volumes, a feature inventory spreadsheet and an executive deck, none of which a markdown-only search returns. Check the git history too, since these are often added and later removed.

They are worth opening even when markdown mirrors exist, because the split is usually consistent and useful: the markdown carries the technical register of routes, models and permissions, while the spreadsheet or deck carries the **business view**, what each module is for and what problem it solves. The second is what intake needs and what the executive summary has to be written from. See `PROBES.md` for extracting them without dependencies.

Two things come out of this, and both are cheap.

**A claimed-feature inventory.** Write down what the project says it has built. This is the list you will test against, and it is more useful than a list you derive yourself, because it captures intent.

**Contradictions inside the documents.** Read them against each other and against themselves. Stale documentation is common; documents that assert both states for the same feature are a stronger finding, because a reader cannot tell which half to distrust. One roadmap encountered in practice stated in its opening line that later phases were "documented only, not built yet", listed twenty-three of those stages as **Built** in a table below, then repeated the "not built" claim underneath the table, and finally declared the whole programme complete. Every claim in that file is now worthless.

### Verify claimed against actual, per feature

For each claimed capability, find the **mechanism**, not the evidence of intent. A feature is not built because a table exists, a screen renders, or an admin page offers a setting.

Ask in this order:

1. Is there a database table? Necessary, meaningless alone.
2. Is anything ever written to it, including by nested writes and seeds?
3. Is there a code path that performs the actual capability, as opposed to recording that it happened?
4. Is there a way for a user to reach that path?

The gap between step 2 and step 3 is where the worst findings live. In one system a "a media-capture and proof-of-delivery" stage marked Built consisted of a table written by one form action taking a filename as a parameter, with no capture, no upload route and a nullable file reference. A proof record could exist pointing at no file. The same system offered single sign-on administration screens with no authentication library installed.

Report these as **claimed but not built**, separately from defects. They are a different problem for the reader: a defect is fixed, an absent capability was budgeted for and does not exist.

## Identify the functional domains

Systems built this way frequently span several domains that would normally be separate products. Enumerate them, because **reliability means something different in each**, and the system needs the strictest applicable standard per domain rather than one standard overall.

| Domain | What reliable means here | Characteristic failure |
|---|---|---|
| **Records and CRM** | Referential integrity, access control, audit trail | Wrong or orphaned data, discovered late |
| **Operations and scheduling** | Concurrency safety, conflict detection, atomic commitment | Double-booking, a slot silently unassigned |
| **Capture and upload** | Durability of in-flight work, resumability, client-side buffering, deploy coordination | Irrecoverable loss of something that cannot be recreated |
| **Evidence and compliance** | Immutability, provenance, chain of custody | Cannot demonstrate an obligation was met |
| **Reporting and analytics** | Reconciliation to source, lineage, period locking | Confidently wrong figures acted upon |
| **External or customer-facing** | Authentication, tenant isolation, rate limiting | Exposure of one client's data to another |

**Capture is the domain most often assessed wrongly,** because it is the only one where the data cannot be recreated. A CRM record entered twice can be entered again. An audio or video recording, a captured signature, or a photograph taken at a site visit cannot. Anything holding irreplaceable data needs durability guarantees that a records system does not, and those guarantees are almost never present unless someone specified them.

**Evidence is the domain most often underrated.** Proof-of-delivery records, compliance attestations and completion certificates carry commercial and sometimes legal weight, and are judged on whether they can be trusted rather than on whether the screen works.

Assign a criticality tier per domain, not per application.

## Set the criticality tier

Everything downstream is calibrated against this. Pick one and say so explicitly in the report.

| Tier | Description | Wrong data costs | Bar |
|---|---|---|---|
| **1. System of record** | Money, inventory, contractual obligations, statutory reporting. Nothing else holds the truth. | Direct revenue loss, restatement, regulatory exposure | Correctness and recoverability are absolute. Silent wrongness is a blocker. Audit trail and change control are in scope. |
| **2. Operational** | Runs a process. Errors cause rework and delay, but the truth is recoverable elsewhere. | Wasted time, missed deadlines, customer irritation | Availability and data integrity matter. Silent wrongness is serious but survivable. |
| **3. Informational** | Notes, comments, internal knowledge, dashboards over data owned elsewhere. | Mild annoyance | Security and privacy still apply. Most correctness and recovery findings drop to backlog. |

**Tier is not one number for the whole system.** A product can hold Tier 1 and Tier 3 modules. Assess per module and say which are which. A comment log attached to a booking ledger inherits none of the ledger's bar; the ledger inherits none of the comment log's tolerance.

## How the tier moves the bar

The finding does not change. Its severity does.

| Finding | Tier 3 | Tier 1 |
|---|---|---|
| No automated backups | Should fix | Blocker |
| Reported figures wrong | May not have figures | Blocker |
| No referential integrity | Backlog | Blocker |
| No audit trail | Backlog | Blocker, and likely an audit finding |
| Cannot revoke sessions promptly | Should fix | Blocker if the data is sensitive |
| No tests | Should fix | Blocker for the repair programme |
| No design system | Cost of ownership | Cost of ownership |

Never quietly apply the Tier 1 bar to a Tier 3 system. Doing so produces a report that reads as alarmist, and the real findings get discounted with the inflated ones.

## Weigh the support model separately

Technical debt is only debt if someone must service it. The same codebase carries different risk depending on who is behind it.

| Support | Effect on the recommendation |
|---|---|
| A team with capacity | Findings become a backlog. Recommend sequencing. |
| One or two engineers | Findings become a queue with a bus factor. Recommend triage and knowledge capture first. |
| An agency or contractor | Response time and knowledge transfer become findings in their own right. Ask what the contract covers. |
| Nobody | Every finding requiring a human to notice or act is effectively unmitigated. This can turn a Tier 2 pass into a fail on its own. |

Ask explicitly what happens when it breaks outside business hours, and whether anyone can currently deploy a fix.

## Establish the domain norms before judging

Do not assess against a generic checklist. Assess against how this class of system is normally built, and be able to say why the norm exists. Absence of an established pattern is a finding; presence of an unusual one needs a reason.

Spend a short pass establishing the norms for the domain, from documentation, comparable open-source systems, or the standards that apply. Then check for them specifically.

Common examples:

- **Ledgers and financial records.** Entries are append-only and corrections are contra entries, never edits. Periods close and lock. Everything reconciles to a source. The reason is that history must be reconstructable and auditors test exactly that.
- **Inventory and stock.** Movements are recorded as transactions and balances are derived, not stored and mutated. The reason is that a mutated balance loses the ability to explain itself and cannot be reconciled after a discrepancy.
- **Booking and scheduling.** Availability checks and commitment are one atomic operation, and overbooking rules are explicit. The reason is that concurrency here creates real liabilities.
- **Anything with imports.** Idempotent, restartable, and reconciled against a source count. The reason is that imports fail halfway and someone will run them twice.
- **Anything with periods.** An explicit stored period on the record, not derived at read time from a timestamp. The reason is that derivation moves figures between periods when a timezone or boundary rule changes.
- **Multi-tenant or customer-facing data.** Scoping enforced at the query layer rather than per endpoint. The reason is that per-endpoint checks fail open the moment somebody adds an endpoint.

When a norm is absent, say what the norm is, why it exists, and what this system does instead. That is far more persuasive than citing a rule, and it tells the reader whether the deviation was reasoned or accidental.

## Record it

Open the assessment with a short context block: tier, what the system does, who uses it, who supports it, and the domain norms it is being measured against. Every severity in the report is then legible as a judgement rather than an assertion, and a reader who disagrees with the tier can see exactly what would change.
