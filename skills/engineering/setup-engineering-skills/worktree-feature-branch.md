# Branching / worktree workflow: feature branch (squash)

A feature is anchored by a single issue — the PRD/parent issue when there is one, or
the lead slice issue when there isn't. **One feature ↔ one branch ↔ one worktree.**

All slice issues for the feature are implemented on that one branch/worktree, not on a
branch each. The branch is squash-merged so the whole feature lands as one summarised
change instead of sprawling into a branch-per-slice.

See `git-worktree-conventions` for branch/worktree naming and the bare-repo layout.

## How slices map to git

- Create one branch + worktree from the anchor issue's number, e.g. branch
  `42-api/feat/billing-overhaul`, worktree `api-feat-billing-overhaul`.
- Implement each slice as one (or a few) commits on that branch — do **not** create a
  branch or worktree per slice.
- Close each slice with a `Closes #<slice>` trailer in its commit message.
- When the feature is done, squash-merge the branch to the default branch. Collect the
  `Closes #...` references for every slice (plus the anchor) into the squash commit
  message / MR description so they all close on merge.

```
PRD issue #42  ──►  branch  42-api/feat/billing-overhaul
                    worktree api-feat-billing-overhaul
                       ├─ commit "add billing schema"      Closes #43
                       ├─ commit "wire invoice endpoint"   Closes #44
                       └─ commit "render invoice table"     Closes #45
                    squash-merge ──► main   (Closes #42, #43, #44, #45)
```

## When to use a standalone worktree

Use a single dedicated worktree for a lone fix/chore/refactor that has no PRD parent —
there the single issue *is* the feature anchor.
