---
name: to-issues
description: Break a plan, spec, or PRD into an ordered implementation checklist of tracer-bullet vertical slices on the project issue tracker. Use when user wants to convert a plan into issues, create implementation tickets, or break down work into issues.
---

# To Issues

Break a plan into a slice checklist using vertical slices (tracer bullets).

How these slices map onto git follows the repo's **branching / worktree workflow** in
`docs/agents/worktree.md` (set up by `/setup-engineering-skills`):

- **Feature branch (the default):** every slice is implemented as plain commits on one
  branch/worktree anchored to the feature (the PRD/parent issue, or the lead slice if
  there is no PRD), each closed with a `Closes #<slice>` trailer, then squash-merged as
  one change. Do **not** create a branch or worktree per slice.
- **Branch per slice:** each slice is independently grabbable and gets its own
  branch/worktree, merged on its own.

Either way the slices themselves are the same; only their git mapping differs. See
`git-worktree-conventions` for naming and folder rules.

The issue tracker and triage label vocabulary should have been provided to you. Run `/setup-engineering-skills` if not.

## Process

### 1. Gather context

Work from whatever is already in the conversation context. If the user passes an issue reference (issue number, URL, or path) as an argument, fetch it from the issue tracker and read its full body and comments.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code. Issue titles and descriptions should use the project's domain glossary vocabulary, and respect ADRs in the area you're touching.

### 3. Draft vertical slices

Break the plan into **tracer bullet** issues. Each issue is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

Slices may be 'HITL' or 'AFK'. HITL slices require human interaction, such as an architectural decision or a design review. AFK slices can be implemented and merged without human interaction. Prefer AFK over HITL where possible.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
</vertical-slice-rules>

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each slice, show:

- **Title**: short descriptive name
- **Order**: where it sits in the implementation sequence
- **Type**: HITL / AFK
- **Blocked by**: which earlier slices (if any) must land first
- **User stories covered**: which user stories this addresses (if the source material has them)

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Is the implementation order / dependency sequence correct?
- Should any slices be merged or split further?
- Are the correct slices marked as HITL and AFK?

Iterate until the user approves the breakdown.

### 5. Publish the issues to the issue tracker

For each approved slice, publish a new issue to the issue tracker. Use the issue body template below. Apply the `needs-triage` triage label so each issue enters the normal triage flow.

Publish issues in dependency order (blockers first) so you can reference real issue identifiers in the "Blocked by" field.

<issue-template>
## Parent

A reference to the parent issue on the issue tracker (if the source was an existing issue, otherwise omit this section).

## What to build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Blocked by

- A reference to the blocking ticket (if any)

Or "None - can start immediately" if no blockers.

</issue-template>

Do NOT close or modify any parent issue.
