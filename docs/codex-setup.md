# Using agent-skills with Codex

This repository is also a [Codex plugin](https://developers.openai.com/codex/plugins/build). The same root-level `skills/` directory used by Claude Code is consumed by Codex, so no files are copied or duplicated.

## Install

```bash
codex plugin marketplace add addyosmani/agent-skills
codex plugin add agent-skills@agent-skills
```

> Requires Codex CLI v0.122 or later. On older releases the command was `codex marketplace add`. See the [Codex CLI docs](https://developers.openai.com/codex/cli).

The first command registers this repository as the `agent-skills` marketplace. The second command installs and enables the `agent-skills` plugin from that marketplace. Start a new Codex session after installation so the skills are discovered.

Local clones work too:

```bash
codex plugin marketplace add /path/to/your/clone
codex plugin add agent-skills@agent-skills
```

## Usage

After install, invoke a skill in Codex chat with `@` (e.g. `@spec` or `@spec-driven-development`) or describe the task and let Codex pick the right skill. The ten short lifecycle skills are `spec`, `plan`, `build`, `test`, `review`, `ship`, `constraints`, `code-simplify`, `webperf`, and `a11y`; other skills remain available by their names under `skills/`.

[Codex uses progressive disclosure](https://developers.openai.com/codex/skills): it starts with each skill's `name` and `description`, chooses skills on demand, then loads the full `SKILL.md` only when selected. Do not also paste `using-agent-skills/SKILL.md` into `AGENTS.md`, a system prompt, or other always-on context: that stacks the pack's meta-router on Codex's native router and adds unnecessary routing work. The meta-skill can remain installed with the pack; the warning is specifically against preloading its full instructions.

## How it works

- `.codex-plugin/plugin.json` — Codex plugin manifest at the repo root. Points `skills` at `./skills/` and provides the metadata required by Codex.
- `.agents/plugins/marketplace.json` — marketplace entry declaring the repo root (`./`) as the plugin source.
- `skills/<name>/SKILL.md` — unchanged. Codex and Claude Code share the same `name` + `description` frontmatter format, so one file serves both platforms.

The `SessionStart` script under `hooks/` is a standalone helper for hosts without native skill routing and is not wired by this plugin. In Codex, invoke the skill directly (for example, `@spec` or `@spec-driven-development`).
