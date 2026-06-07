# Claude Code setup

This library installs into Claude Code as a plugin. Two install paths: local development (point Claude at the cloned repo) or marketplace (push the repo to GitHub and install via the marketplace command).

## Local install (recommended for personal use)

Clone the repo wherever you keep your tools, then point Claude Code at the cloned directory.

```bash
git clone <your-fork-url> ~/dev/skills
claude --plugin-dir ~/dev/skills
```

Claude Code reads `.claude-plugin/plugin.json` from that directory and registers every skill listed there. Skill descriptions become trigger surfaces, so the agent picks the right skill based on what you ask.

If you want the plugin to load by default, add the `--plugin-dir` flag to your Claude Code launch script or shell alias.

## Marketplace install (for sharing)

If you push the repo to GitHub, anyone can install via Claude Code's marketplace.

1. Add a `marketplace.json` to `.claude-plugin/` that references this plugin (see [`addyosmani/agent-skills`](https://github.com/addyosmani/agent-skills/blob/main/.claude-plugin/marketplace.json) for a working example).
2. Push the repo.
3. Users install with:

```
/plugin marketplace add <your-username>/<repo>
/plugin install adrian-skills@<marketplace-name>
```

This library currently ships only `plugin.json`, not `marketplace.json`. Local install works without the marketplace file.

## How it works once installed

Skills are loaded on demand. Claude Code reads only the frontmatter (`name` and `description`) at startup. The full `SKILL.md` body loads into context when the agent decides the skill applies, based on the description matching the user's request.

That means descriptions must be precise. The library's skills follow a consistent format: what the skill does in third person, then explicit `Use when...` trigger phrases. See `productivity/write-a-skill` for the format.

## Updating

Skills update by pulling the repo. There is no separate version step.

```bash
cd ~/dev/skills && git pull
```

Restart Claude Code to pick up changes to `plugin.json`. Changes inside individual `SKILL.md` files take effect on the next session that loads the skill.

## Adding a new skill

1. Create the directory: `skills/<bucket>/<skill-name>/SKILL.md`.
2. Write the skill following `productivity/write-a-skill`.
3. Add the path to `.claude-plugin/plugin.json` under `skills`.
4. Add the entry to the bucket's `README.md`.
5. Restart Claude Code.

## Removing a skill

1. Remove the path from `.claude-plugin/plugin.json`.
2. Remove the entry from the bucket's `README.md`.
3. Either delete the skill directory or move it to `skills/deprecated/` (kept around but unlinked from the plugin manifest).
