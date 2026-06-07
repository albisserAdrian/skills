# Branching / worktree workflow: branch per slice

Each slice issue is independently grabbable: it gets its **own branch and its own
worktree**, and is merged on its own. Use this when slices are worked in parallel —
for example several AFK agents on the same feature at once — and you accept more
branches and a noisier history in exchange.

See `git-worktree-conventions` for branch/worktree naming and the bare-repo layout.

## How slices map to git

- Create one branch + worktree per slice issue, keyed to that slice's number, e.g.
  branch `43-api/feat/billing-schema`, worktree `api-feat-billing-schema`.
- Implement and verify the slice on its own branch.
- Close the slice with a `Closes #<slice>` trailer so its own merge closes it.
- Merge each slice independently (squash or not, per repo norm). The PRD/parent issue
  closes once its last slice merges — close it manually if your tracker doesn't.

```
PRD issue #42
   ├─ branch 43-api/feat/billing-schema   worktree api-feat-billing-schema   ──► main  (Closes #43)
   ├─ branch 44-api/feat/invoice-endpoint worktree api-feat-invoice-endpoint ──► main  (Closes #44)
   └─ branch 45-web/feat/invoice-table    worktree web-feat-invoice-table    ──► main  (Closes #45)
```

## Dependencies

Respect the `Blocked by` order from `to-issues`: a slice that depends on another should
branch off after its blocker merges (or stack on the blocker's branch) to avoid
conflicts.
