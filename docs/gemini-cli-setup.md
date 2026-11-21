# Using agent-skills with Gemini CLI

## Setup

### Option 1: Install as Skills (Recommended)

Gemini CLI has a native skills system that auto-discovers `SKILL.md` files in `.gemini/skills/` or `.agents/skills/` directories. Each skill activates on demand when it matches your task.

**Install from the repo:**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**Or install from a local clone:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
gemini skills install /path/to/agent-skills/skills/
```

**Install for a specific workspace only:**

```bash
gemini skills install /path/to/agent-skills/skills/ --scope workspace
```

Skills installed at workspace scope go into `.gemini/skills/` (or `.agents/skills/`). User-level skills go into `~/.gemini/skills/`.

Once installed, verify with:

```
/skills list
```

Gemini CLI injects skill names and descriptions into the prompt automatically. When it recognizes a matching task, it asks permission to activate the skill before loading its full instructions.

### Option 2: GEMINI.md (Persistent Context)

For skills you want always loaded as persistent project context (rather than on-demand activation), add them to your project's `GEMINI.md`:

```bash
# Create GEMINI.md with core skills as persistent context
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md > GEMINI.md
echo -e "\n---\n" >> GEMINI.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> GEMINI.md
```

You can also modularize by importing from separate files:

```markdown
# Project Instructions

@skills/test-driven-development/SKILL.md
@skills/incremental-implementation/SKILL.md
```

Use `/memory show` to verify loaded context, and `/memory reload` to refresh after changes.

> **Skills vs GEMINI.md:** Skills are on-demand expertise that activate only when relevant, keeping your context window clean. GEMINI.md provides persistent context loaded for every prompt. Use skills for phase-specific workflows and GEMINI.md for always-on project conventions.

## Recommended Configuration

### Always-On (GEMINI.md)

Add these as persistent context for every session:

- `incremental-implementation` — Build in small verifiable slices
- `code-review-and-quality` — Five-axis review

### On-Demand (Skills)

Install these as skills so they activate only when relevant:

- `test-driven-development` — Activates when implementing logic or fixing bugs
- `spec-driven-development` — Activates when starting a new project or feature
- `frontend-ui-engineering` — Activates when building UI
- `security-and-hardening` — Activates during security reviews
- `performance-optimization` — Activates during performance work

## Advanced Configuration

### MCP Integration

Many skills in this pack leverage [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) tools to interact with the environment. For example:

- `browser-testing-with-devtools` uses the `chrome-devtools` MCP extension.
- `performance-optimization` can benefit from performance-related MCP tools.

To enable these, configure the relevant MCP servers under `mcpServers` in your Gemini CLI settings (`~/.gemini/settings.json`, or `.gemini/settings.json` in a project).

### Session Hooks

Gemini CLI supports session lifecycle hooks. You can use these to automatically inject context or run validation scripts at the start of a session.

To replicate the `agent-skills` experience from other tools, you can configure a `SessionStart` hook that reminds you of the available skills or loads a meta-skill. `hooks/session-start.sh` in this repo is a ready-made script for that: it prints the `using-agent-skills` meta-skill as a standard `SessionStart` JSON envelope on stdout.

## Explicit Context Loading

To request a particular workflow, describe it by name in natural language. Gemini can select the matching skill from the available catalog.

## Workflow Skills

Lifecycle workflows are installed as skills alongside specialist skills. Describe the workflow you want in natural language, and Gemini CLI can select a relevant skill. Use `/skills list` to inspect discovered skills.

## Usage Tips

1. **Prefer skills over GEMINI.md** — Skills activate on demand and keep your context window focused. Only put instructions in GEMINI.md if you want them always loaded.
2. **Skill descriptions matter** — The `description` field in each skill's frontmatter guides auto-discovery.
3. **Use agents for review** — Copy `agents/code-reviewer.md` content when requesting structured code reviews.
4. **Combine with references** — Reference checklists from `references/` when working on specific quality areas like testing or performance.

