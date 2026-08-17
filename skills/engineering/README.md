# Engineering

Skills I use daily for code work.

> **Start here in each repo:** run [`setup-engineering-skills`](./setup-engineering-skills/SKILL.md) once before using the others. It records this repo's issue tracker, triage labels, domain doc layout, and branching/worktree workflow into `docs/agents/` so `to-prd`, `to-issues`, `triage`, and the rest behave correctly here. The flow-oriented skills (`to-prd`, `to-issues`, `triage`) will tell you to run it if it hasn't been.

- **[start](./start/SKILL.md)**: originate new work safely: create the anchor issue and a branch/worktree off `main` before any files are written, then hand off to `grill-with-docs`.
- **[diagnose](./diagnose/SKILL.md)**: disciplined diagnosis loop for hard bugs and performance regressions: reproduce → minimise → hypothesise → instrument → fix → regression-test.
- **[grill-with-docs](./grill-with-docs/SKILL.md)**: grilling session that challenges your plan against the existing domain model, sharpens terminology, and updates `CONTEXT.md` and ADRs inline.
- **[triage](./triage/SKILL.md)**: triage issues through a state machine of triage roles.
- **[improve-codebase-architecture](./improve-codebase-architecture/SKILL.md)**: find deepening opportunities in a codebase, informed by the domain language in `CONTEXT.md` and the decisions in `docs/adr/`.
- **[review](./review/SKILL.md)**: five-axis code review across correctness, readability, architecture, security, and performance, with severity-tagged findings.
- **[assess-codebase](./assess-codebase/SKILL.md)**: whole-system assessment of an existing codebase for production fitness, across security, data integrity, correctness, platform, practice and governance. Produces an executive summary for non-engineers alongside the technical report. Use `review` for a single change.
- **[security](./security/SKILL.md)**: security playbook organised as three tiers (always-do, ask-first, never-do) plus an OWASP-shaped issue list and dependency-vulnerability triage tree.
- **[setup-engineering-skills](./setup-engineering-skills/SKILL.md)**: scaffolds the `## Agent skills` block in CLAUDE.md/AGENTS.md and `docs/agents/` so the engineering skills know this repo's issue tracker, triage labels, domain doc layout, and branching/worktree workflow. Run before first use of `to-prd`, `to-issues`, `triage`, etc.
- **[code-simplification](./code-simplification/SKILL.md)**: simplify code for clarity while preserving exact behaviour. Covers expression-level cleanup; structural changes belong in `improve-codebase-architecture`.
- **[documentation](./documentation/SKILL.md)**: document the why, not the what. Inline comments, API docs, READMEs, changelogs, and agent-context files. Defers to `grill-with-docs/ADR-FORMAT.md` for ADRs.
- **[tdd](./tdd/SKILL.md)**: test-driven development with a red-green-refactor loop. Builds features or fixes bugs one vertical slice at a time.
- **[to-issues](./to-issues/SKILL.md)**: break any plan, spec, or PRD into an ordered checklist of vertical-slice issues; their git mapping follows the repo's branching workflow (`docs/agents/worktree.md`).
- **[to-prd](./to-prd/SKILL.md)**: turn the current conversation context into a PRD and submit it as a GitHub issue.
- **[zoom-out](./zoom-out/SKILL.md)**: tell the agent to zoom out and give broader context or a higher-level perspective on an unfamiliar section of code.
