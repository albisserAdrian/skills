---
name: start
description: Originate new work safely by creating the anchor (umbrella) issue on the project issue tracker first, then a worktree or branch off main, before any files are written. Use when the user wants to start a new feature, fix, or chore; kick off new work; begin an issue; or says "start" / "let's start something new". Run before grill-with-docs so its CONTEXT.md/ADR edits never land on main.
---

# Start

Originate a new unit of work. The job is to get an **anchor issue** and a **clean branch
or worktree** in place *before* anything writes files, so `grill-with-docs`, `to-prd`,
and friends never scribble on `main`.

The order matters: issue first, then branch/worktree, then hand off. Do not skip ahead.

The issue tracker, triage label vocabulary, and branching/worktree workflow should have
been provided to you (see `docs/agents/`). Run `/setup-engineering-skills` if not.

## Process

### 1. Get the title and type

Ask for two things (one message, both at once):

- **Title**: one line, what the work is.
- **Type**: one of `feat`, `fix`, `chore`, `docs`, `refactor`. This becomes the
  `<type>` segment of the branch name and hints the triage category.

Nothing else. The detail is what `grill-with-docs` is for; don't interview here.

### 2. Create the anchor issue

This is the **feature anchor**, the umbrella issue that `to-prd` slices hang off and
whose number names the branch/worktree. Create it on the project issue tracker (per
`docs/agents/issue-tracker.md`) with the title, a one-line stub body noting it's a freshly
started anchor to be fleshed out, and the `needs-triage` label.

Capture the **issue number** from the CLI output; you need it for the branch name.

> Do not create files in the repo yet. Only the issue exists at this point.

### 3. Detect the local setup: worktree or plain branch

Run `git worktree list` and look for a bare layout (`.bare/` + flat sibling worktrees, or
more than one worktree entry):

- **Worktree layout** → create a new worktree.
- **Plain single working dir** → create a plain branch with `git checkout -b` off the
  default branch.

Either way, name it per `git-worktree-conventions`, keyed to the anchor issue number.
Decide single-app vs monorepo (lead with the app/package name only in a monorepo).
**Propose the branch/worktree name and confirm with the user before creating it.**
Creating branches and worktrees is outward-facing, so confirm once.

Composition (from `git-worktree-conventions`):

| Layout      | Branch                          | Worktree dir            |
| ----------- | ------------------------------- | ----------------------- |
| Single-app  | `<issue>-<type>/<desc>`         | `<type>-<desc>`         |
| Monorepo    | `<issue>-<app>/<type>/<desc>`   | `<app>-<type>-<desc>`   |

```bash
# worktree layout (flat dir name, namespaced branch, kept decoupled)
git worktree add <type>-<desc> -b <issue>-<type>/<desc>

# plain-branch repo
git checkout -b <issue>-<type>/<desc>
```

### 4. Print the access command and open tmux

Print the command the user runs to get into the work:

- Worktree → `cd <absolute-worktree-path>`
- Plain branch → already checked out in the current dir (`git checkout <branch>` to return).

If running inside tmux (`$TMUX` is set), open a new window there pointed at the work, named
after the branch's short description:

```bash
[ -n "$TMUX" ] && tmux new-window -c "<worktree-path-or-repo-root>" -n "<desc>"
```

Tell the user the window name so they can find it.

### 5. Hand off to grill-with-docs

Work now happens **inside the new worktree/branch**, so its file edits stay off `main`.
Hand off to `grill-with-docs` to interview and shape the plan, writing `CONTEXT.md` and
ADRs on the branch.

For a simple change with no PRD, skip the PRD: grill if useful, then implement directly.
The anchor issue *is* the feature anchor for the lone fix/chore.

## Handoff to downstream skills

The issue created here is the feature anchor. When `to-prd` or `to-issues` run later, they
should fill in / hang slices off **this** issue, not open a second anchor.
