# Using agent-skills with GitHub Copilot CLI

The standalone `copilot` command-line tool installs this repository as a plugin and discovers every skill in `skills/`. For Copilot inside VS Code, see [copilot-setup.md](copilot-setup.md) instead — the setup and the invocation model are different.

## Install

**From this repository's marketplace** — register it, then install from it. `addy-agent-skills` is the marketplace name this repository declares, not a GitHub-wide registry, and the name only resolves after the `marketplace add`:

```bash
copilot plugin marketplace add addyosmani/agent-skills
copilot plugin install agent-skills@addy-agent-skills
```

**Directly from the repository**, without registering a marketplace:

```bash
copilot plugin install addyosmani/agent-skills
```

**From a local clone**, for a session-scoped development install:

```bash
git clone https://github.com/addyosmani/agent-skills.git
copilot --plugin-dir /path/to/agent-skills
```

`--plugin-dir` loads the plugin for that session only and installs nothing persistently — use it while editing skills.

## Verify

```bash
copilot plugin list   # agent-skills@addy-agent-skills
copilot skill list    # the plugin's skills, alongside the built-in ones
```

In an interactive session, `/skills list` shows the same catalog.

## What you get, and what you don't

The root `plugin.json` is the manifest Copilot CLI reads. It takes precedence over `.claude-plugin/plugin.json`, which belongs to Claude Code. Its `$schema` line declares an [Agent Plugins 1.0.0](https://agent-plugins.org) manifest, which switches the CLI into Agent Plugins mode. The manifest sets no component paths, so they fall back to their defaults:

- **Skills: available.** With no explicit path, the CLI uses the conventional `skills/` directory, and the skills are discovered there. Lifecycle workflows are skills, not command wrappers.
- **Commands: not available.** In Agent Plugins mode the CLI reads commands only from `com.github.copilot/commands/`, which this repo doesn't have, so no separate command files register. Describe the workflow you want in natural language to use its skill.
For manifest precedence and the per-component path defaults, see the [CLI plugin reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-plugin-reference) and [Creating plugins](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-creating).

## Usage

Describe the task or workflow you want, and Copilot can select the relevant skill:

> Write a spec for [the feature].

> Write a failing test for this bug first, then fix it.

> Review my staged changes.

## Troubleshooting

| Symptom | What to check |
|---------|---------------|
| `plugin install` can't resolve `agent-skills@addy-agent-skills` | Run `copilot plugin marketplace add addyosmani/agent-skills` first — that name only resolves once the marketplace is registered. Or install the repository directly. |
| Plugin installed but no skills | `copilot plugin list` to confirm the plugin, then `copilot skill list` (or `/skills list` in session) to see what was discovered. If the list is empty, check the root `plugin.json` against the [1.0.0 schema](https://agent-plugins.org/schemas/1.0.0/plugin.schema.json): in Agent Plugins mode any schema error (say, `author` as a string instead of an object) is fatal and the CLI loads no skills. |
| Skill not selected as expected | Check `copilot skill list` or `/skills list`; describe the desired workflow in natural language. |
| Skills changed locally but the CLI shows the old copy | Start a fresh session, or run with `--plugin-dir` pointing at your clone. |
