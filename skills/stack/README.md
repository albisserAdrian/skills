# Stack

Language- and library-specific code-writing rules. These fire when working in a particular tech stack, as opposed to the codebase-agnostic guidance in `engineering/`.

- **[avoid-use-effect](./avoid-use-effect/SKILL.md)**: React useEffect avoidance guide. Prefer ref callbacks, event handlers with `flushSync`, CSS, and `useSyncExternalStore` over reactive effects.
- **[api-and-interface-design](./api-and-interface-design/SKILL.md)**: reference for stable API and module-boundary design (REST, GraphQL, type contracts, frontend/backend boundaries). Kept around for when designing public interfaces.
- **[frontend-ui-engineering](./frontend-ui-engineering/SKILL.md)**: reference for production-quality UI work (components, layouts, state, accessibility). Kept around for when building user-facing interfaces.
- **[ci-cd-and-automation](./ci-cd-and-automation/SKILL.md)**: reference for CI/CD pipeline setup, quality gates, and deployment strategies. Kept around for when configuring build and deploy automation.
- **[git-worktree-conventions](./git-worktree-conventions/SKILL.md)**: conventions for git worktree folder structure and branch naming — bare-repo layout, monorepo app/package-first naming, GitLab issue linking, and decoupling worktree path from branch name. Kept around for when setting up worktrees or naming branches.

