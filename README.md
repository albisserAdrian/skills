# Skills

Personal collection of agent skills for Claude Code, OpenCode, and any other agent that reads a rules file. Built by trimming and adapting [`mattpocock/skills`](https://github.com/mattpocock/skills) and [`addyosmani/agent-skills`](https://github.com/addyosmani/agent-skills) into a smaller, opinionated set tuned to my own workflow.

## Buckets

- **[engineering/](./skills/engineering/README.md)**: codebase-agnostic process and discipline. Daily-use skills for review, security, tdd, debugging, refactoring, documentation, and issue workflow.
- **[stack/](./skills/stack/README.md)**: language and library-specific code-writing rules. React, API design, frontend, CI/CD.
- **[productivity/](./skills/productivity/README.md)**: non-code workflow tools. Writing-style enforcement, idea grilling, skill authoring, communication compression.
- **[misc/](./skills/misc/README.md)**: rarely-used reference material kept around for specific occasions.

## Install

| Tool | Setup |
|---|---|
| Claude Code | [docs/claude-code-setup.md](./docs/claude-code-setup.md) |
| OpenCode | [docs/opencode-setup.md](./docs/opencode-setup.md) |
| Other agents (Cursor, Copilot, Gemini, Windsurf) | Point at `skills/` as reference material; use the intent mapping in `AGENTS.md` to decide which skill applies. |

The shortest path:

```bash
git clone <this-repo-url> ~/dev/skills

# Claude Code
claude --plugin-dir ~/dev/skills

# OpenCode (from a workspace)
ln -s ~/dev/skills/AGENTS.md AGENTS.md
```

## How skills are loaded

Each `SKILL.md` has YAML frontmatter with `name` and `description`. The description is the only thing the agent sees when deciding whether to load the skill, so triggers are explicit (`Use when...` phrases naming concrete signals).

Skills are loaded on demand. The agent reads only descriptions at startup; the body loads into context when the skill is invoked. That keeps the working context clean even when many skills are registered.

## Adding a new skill

Use the `productivity/write-a-skill` skill. The short version:

1. Create `skills/<bucket>/<skill-name>/SKILL.md` with frontmatter (`name`, `description`) and a body that follows the bucket's house style.
2. Add the path to `.claude-plugin/plugin.json` (for Claude Code).
3. Add an intent row to `AGENTS.md` (for OpenCode and other agent-rule-file tools).
4. Add the skill to the bucket's `README.md`.

## House style

All prose in this repo follows `productivity/natural-writing`:

- No em dashes as punctuation. Use comma, colon, period, or parentheses.
- Sentence case headings.
- No AI-tell vocabulary (delve, tapestry, intricate, pivotal, etc.).
- Repeat words when clarity demands it; do not rotate synonyms for variation's sake.

Skill descriptions follow the format documented in `productivity/write-a-skill`: third person, what-it-does first, then explicit `Use when...` triggers.

## Credits

This library is a curated remix:

- [`mattpocock/skills`](https://github.com/mattpocock/skills): the lighter base, source of `diagnose`, `tdd`, `triage`, `to-prd`, `to-issues`, `grill-with-docs`, `improve-codebase-architecture`, `caveman`, `grill-me`, `write-a-skill`, `git-guardrails-claude-code`, `migrate-to-shoehorn`, `zoom-out`.
- [`addyosmani/agent-skills`](https://github.com/addyosmani/agent-skills): the heavier reference, source of `review`, `security`, `code-simplification`, `documentation`, `api-and-interface-design`, `frontend-ui-engineering`, `ci-cd-and-automation` (trimmed and adapted to house style).
- Personal additions: `natural-writing`, `avoid-use-effect`.

Original repos:

- [`mattpocock/skills`](https://github.com/mattpocock/skills)
- [`addyosmani/agent-skills`](https://github.com/addyosmani/agent-skills)
