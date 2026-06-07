---
name: git-worktree-conventions
description: Conventions for git worktree folder structure and branch naming, including bare-repo layout, monorepo app/package-first naming, GitLab issue linking, and decoupling worktree path from branch name. Use when setting up git worktrees, creating a branch or worktree, naming branches/worktrees, working in a bare-repo or monorepo, or deciding folder structure for feature/fix/chore work.
---

# Git Worktree & Branch Conventions

## Core rule

**One feature ↔ one branch ↔ one worktree.** A feature is anchored by a single
GitLab issue — usually a PRD/parent issue. All the work for that feature lands on
that one branch and worktree, so it can be squash-merged as one summarised change
instead of sprawling into a branch-per-slice.

This is the **default (feature-branch) workflow**. A repo can instead choose the
**branch-per-slice** workflow, where each slice issue gets its own branch and worktree
for parallel work. The repo's choice is recorded in `docs/agents/worktree.md` (set up
by `/setup-engineering-skills`) — read it first; it's authoritative. The naming and
folder rules below apply to both.

**The worktree directory name and the git branch name are decoupled.** Pass them as
separate arguments so you keep scannable flat folders *and* namespaced branches:

```bash
git worktree add <flat-dir-name> <branch-name>        # existing branch
git worktree add <flat-dir-name> -b <branch-name>     # new branch
```

The issue number in the branch name is the **feature anchor** issue (the PRD/parent
when there is one). The issue number lives in the **branch name**, never in the
directory name.

## Feature worktree vs task worktree

Under the default feature-branch workflow, when a PRD has been broken into slice issues
(see `to-issues`), do **not** create a branch or worktree per slice. Create one feature
worktree keyed to the PRD issue and implement every slice as plain commits on that
branch (for the branch-per-slice alternative, see `docs/agents/worktree.md`):

- Each slice is one (or a few) commits on the feature branch.
- Close each slice issue with a `Closes #<slice>` trailer in its commit message.
- When the feature is done, squash-merge the branch to the default branch. Collect the
  `Closes #...` references for every slice (plus the PRD) into the squash commit
  message / MR description so they all close on merge.

```
PRD issue #42  ──►  branch  42-api/feat/billing-overhaul
                    worktree api-feat-billing-overhaul
                       ├─ commit "add billing schema"     Closes #43
                       ├─ commit "wire invoice endpoint"   Closes #44
                       └─ commit "render invoice table"     Closes #45
                    squash-merge ──► main   (Closes #42, #43, #44, #45)
```

Use a standalone **task worktree** only for a lone fix/chore/refactor that has no PRD
parent — there the single issue *is* the feature anchor.

If you skipped the PRD and went straight to slice issues, the lead (or sole) issue
becomes the feature anchor and owns the single worktree.

## Bare-repo layout

Use a single project folder containing the bare clone plus flat sibling worktrees.
Never nest a worktree inside another worktree — it breaks `.git` resolution.

```
~/dev/projects/myapp/
├── .bare/                    # bare repo (git internals only)
├── .git                      # a *file* containing: gitdir: ./.bare
├── main/                     # worktree for main
├── api-fix-some-description/ # worktree
└── web-chore-deps/           # worktree
```

Setup:

```bash
git clone --bare git@gitlab.com:you/myapp.git myapp/.bare
cd myapp
echo "gitdir: ./.bare" > .git
git config remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'
git fetch origin
git worktree add main main
```

## Naming

### Monorepo (multiple apps/packages)

Lead with the **app/package name**, then the **type** (`fix`, `feat`, `chore`,
`refactor`, etc.), then a **short description**. This makes a flat `ls` scannable.

| Part        | Branch (with issue #, slashes) | Worktree dir (no #, hyphens) |
| ----------- | ------------------------------ | ---------------------------- |
| Example     | `31-api/fix/some-description`  | `api-fix-some-description`    |
| Composition | `<issue>-<app>/<type>/<desc>`  | `<app>-<type>-<desc>`         |

```bash
git worktree add api-fix-some-description -b 31-api/fix/some-description
```

The branch keeps the issue number and slashes (`31-api/fix/some-description`); the
directory drops the number and flattens slashes to hyphens (`api-fix-some-description`).

### Single-app repo

Drop the app/package prefix: branch `31-fix/some-description`, worktree dir
`fix-some-description`.

## Branch-only repos (no worktrees)

When a repo uses plain branches (no worktrees), there is no directory to name, so use
the **branch name with the issue number and slashes**:

```bash
git checkout -b 31-api/fix/some-description
```

Same composition as the worktree branch — `<issue>-<app>/<type>/<desc>` for a monorepo,
`<issue>-<type>/<desc>` for a single app.

## Conventions

- Use lowercase and hyphens in descriptions; avoid spaces and special characters.
- Keep descriptions short but recognizable: `api-fix-auth` beats `api-fix-the-auth-bug-from-monday`.
- Remove worktrees with `git worktree remove <dir>`, never `rm -rf`.
- Worktrees share the object database but **not** the working dir — `node_modules`/build
  artifacts are per-worktree. In a monorepo, reinstall deps (or use a shared pnpm store)
  per worktree.
- Default workflow: one feature (PRD/anchor issue) ↔ one branch ↔ one worktree, with
  slice issues as commits on that branch — not their own branches or worktrees. The
  branch-per-slice alternative is opt-in via `docs/agents/worktree.md`.
