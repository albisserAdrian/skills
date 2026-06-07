# AGENTS.md

Guidance for AI coding agents (Claude Code, OpenCode, Cursor, Copilot, etc.) working in this repository.

## Repository overview

A personal library of agent skills, organised into four buckets:

- `skills/engineering/`: codebase-agnostic process and discipline (review, security, tdd, diagnose, code-simplification, documentation, etc.)
- `skills/stack/`: language and library-specific code-writing rules (React `useEffect` guidance, API design, frontend UI, CI/CD)
- `skills/productivity/`: non-code workflow tools (writing-style enforcement, idea grilling, skill authoring)
- `skills/misc/`: rarely-used reference material kept around for specific occasions

## Skill execution model

When a user request matches a skill, the agent must invoke it rather than implement directly. Skills encode the workflow, exit criteria, and known anti-patterns for that kind of work.

### Core rules

1. If a request matches a skill, you MUST invoke it.
2. Skills live at `skills/<bucket>/<skill-name>/SKILL.md`.
3. Follow the skill instructions exactly. Do not partially apply them.
4. If multiple skills apply, run them in the order listed in the intent mapping below.
5. Never skip a skill's verification step.

### Intent to skill mapping

| User intent or signal | Skill |
|---|---|
| Hard bug, crash, failing test, performance regression | `diagnose` |
| Test-first development, red-green-refactor, building a feature | `tdd` |
| Code review, second opinion before merge, evaluating another agent's output | `review` |
| Anything touching user input, auth, sessions, secrets, file uploads, external APIs | `security` |
| Cleaning up working code for clarity (no behaviour change) | `code-simplification` |
| Structural / architectural change, finding deepening opportunities | `improve-codebase-architecture` |
| Stress-testing a plan against the project's domain model and ADRs | `grill-with-docs` |
| Triaging issues on the issue tracker | `triage` |
| Turning conversation context into a PRD | `to-prd` |
| Breaking a plan into independently-grabbable issues | `to-issues` |
| Inline comments, README, API docs, changelog, agent-context files | `documentation` |
| Zoom out for higher-level view of unfamiliar code | `zoom-out` |
| Writing or editing React components in `.tsx` / `.jsx` (especially `useEffect`) | `stack/avoid-use-effect` |
| Designing an API, public endpoint, module interface, breaking change | `stack/api-and-interface-design` |
| Building or modifying user-facing UI | `stack/frontend-ui-engineering` |
| Setting up or modifying a CI/CD pipeline | `stack/ci-cd-and-automation` |
| Producing markdown, documentation, READMEs, articles, prose | `productivity/natural-writing` |
| Stress-testing a non-code idea or plan | `productivity/grill-me` |
| Compressing communication to save tokens | `productivity/caveman` |
| Authoring a new skill | `productivity/write-a-skill` |

### Lifecycle mapping

When the user is moving through the development lifecycle without naming a specific skill, follow this implicit ordering:

- DEFINE: `to-prd` (if context warrants it) or `grill-with-docs`
- PLAN: `to-issues`
- BUILD: `tdd` (always when writing code), plus any relevant `stack/*` rule
- VERIFY: `diagnose` (when something is broken), `review` (before declaring done)
- HARDEN: `security` (any time external input or auth is touched)
- DOCUMENT: `documentation` (when shipping a feature or making a decision worth recording)

### Anti-rationalization

Reject these justifications for skipping skills:

- "This is too small for a skill"
- "I can just quickly implement this"
- "I'll gather context first"
- "The skill is overkill here"

Correct behaviour: check for a matching skill first. If one applies, use it.

## Tool-specific notes

### Claude Code
Skills are auto-discovered via `.claude-plugin/plugin.json`. See `docs/claude-code-setup.md`.

### OpenCode
The `skill` tool plus this `AGENTS.md` drives auto-invocation. The `.opencode/skills` symlink exposes the same skill directory. See `docs/opencode-setup.md`.

### Other agents
Any agent that reads a rules file (Cursor, Copilot, Gemini, Windsurf) can be pointed at `skills/` as reference material. The intent mapping above tells it which skill to load for which request.

## Conventions

- Every skill lives at `skills/<bucket>/<name>/SKILL.md`.
- Frontmatter has `name` and `description`. Description starts with what the skill does (third person), then includes one or more `Use when...` trigger phrases.
- Prose in skills follows the rules in `productivity/natural-writing`: no em dashes as punctuation, sentence case headings, no AI-tell vocabulary.
- Bucket READMEs list each skill with a one-line description, with the skill name linked to its `SKILL.md`.
