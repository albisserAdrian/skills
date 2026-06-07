# OpenCode setup

OpenCode does not have a plugin system or auto-discovery for skill directories. Instead, the integration relies on three things working together:

1. The `AGENTS.md` at the repo root, which tells the agent to invoke skills automatically based on user intent.
2. The `.opencode/skills` symlink to `skills/`, which puts every skill where OpenCode's `skill` tool can find it.
3. The skill descriptions themselves, which let the agent decide which skill applies.

This produces an agent-driven workflow that mirrors Claude Code's auto-routing without needing slash commands.

## Install

Clone the repo somewhere OpenCode can read it (typically inside the workspace you're working in, or as a sibling directory).

```bash
git clone <your-fork-url> skills
```

Make sure the agent sees `AGENTS.md` and the `.opencode/skills` symlink. If you cloned into a sibling directory, you may want to symlink or copy `AGENTS.md` into the workspace root.

```bash
ln -s ../skills/AGENTS.md AGENTS.md
```

That's it. No plugin install, no command registration.

## How skill invocation works

For every user request, the agent should:

1. Read `AGENTS.md` (loaded automatically by OpenCode at session start).
2. Match the request against the intent table in `AGENTS.md`.
3. Invoke the matching skill via the `skill` tool.
4. Follow the skill's workflow exactly, including verification steps.
5. Only proceed to direct implementation if no skill matched.

Examples:

- "Fix this 500 error" → `engineering/diagnose`
- "Add user signup" → `engineering/tdd` plus any relevant `stack/*` skill
- "Review this branch" → `engineering/review`
- "Write a README for this project" → `productivity/natural-writing` plus `engineering/documentation`

## Lifecycle without slash commands

OpenCode does not support `/spec`, `/plan`, `/ship`-style commands. The lifecycle is encoded implicitly in `AGENTS.md`:

- DEFINE: `to-prd` or `grill-with-docs`
- PLAN: `to-issues`
- BUILD: `tdd` plus any `stack/*` skill that fits
- VERIFY: `diagnose` for failures, `review` before declaring done
- HARDEN: `security` whenever external input or auth is touched
- DOCUMENT: `documentation` when shipping a feature or recording a decision

The agent walks this lifecycle on its own based on what the user is asking for.

## Limitations

- Skill invocation depends on the model's compliance with `AGENTS.md`. If the agent skips skills, the rules need to be reinforced (either in the prompt or in `AGENTS.md` itself).
- No marketplace install. The repo is the source of truth.
- No automatic update. Pull the repo manually.

## Updating

```bash
cd skills && git pull
```

Changes to `AGENTS.md` take effect on the next session start. Changes inside individual `SKILL.md` files take effect when the skill is next invoked.

## Adding a new skill

1. Create the directory: `skills/<bucket>/<skill-name>/SKILL.md`.
2. Write the skill following `productivity/write-a-skill`.
3. Add an intent row to the table in `AGENTS.md` so the agent knows when to invoke it.
4. Add the entry to the bucket's `README.md`.

The `.opencode/skills` symlink picks up the new skill automatically because it points at `skills/`.
