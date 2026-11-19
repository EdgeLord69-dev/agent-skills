# Using agent-skills with Antigravity CLI (agy)

The `agent-skills` package can be installed as a native plugin in the Antigravity CLI (`agy`), giving the agent access to structured workflows and personas.

## Setup

### Option 1: Native Plugin Installation (Recommended)

Antigravity CLI has a [plugin system](https://www.agy.dev/docs/plugins/) that registers skills and agents. The lifecycle workflows are now skills in `skills/`; use the skills directly rather than relying on legacy command TOMLs.

**Install from the remote repository:**

```bash
agy plugin install https://github.com/addyosmani/agent-skills.git
```

**Install from a local clone:**

1. Clone the repository:
   ```bash
   git clone https://github.com/addyosmani/agent-skills.git
   ```
2. Install the plugin using `agy`:
   ```bash
   agy plugin install /path/to/agent-skills
   ```

This will validate the plugin and install it into your global Antigravity configuration directory (`~/.gemini/config/plugins/agent-skills/`).

> **Note:** on current agy releases the plugin lands under `~/.gemini/config/plugins/`, not the legacy `~/.gemini/antigravity-cli/plugins/` path used by older versions. If you don't see the plugin at the legacy path, check `~/.gemini/config/plugins/agent-skills/` first.

### Option 2: Import from Gemini CLI

If you have already installed `agent-skills` under your legacy Gemini CLI installation, you can import it directly:
```bash
agy plugin import gemini
```

Once installed, verify the active plugin:
```bash
agy plugin list
```

---

## Lifecycle Skills

The ten lifecycle workflows are installed as skills. Invoke them from the plugin's namespaced skill catalog:

| Workflow | Skill |
|----------|-------|
| Define | `/agent-skills:spec` |
| Plan | `/agent-skills:plan` |
| Build | `/agent-skills:build` |
| Verify | `/agent-skills:test` |
| Review | `/agent-skills:review` |
| Simplify | `/agent-skills:code-simplify` |
| Ship | `/agent-skills:ship` |
| Constraints | `/agent-skills:constraints` |
| Web performance | `/agent-skills:webperf` |
| Accessibility | `/agent-skills:a11y` |

Antigravity may reserve names for built-in commands, and skill invocation/catalog behavior can differ by release. Check `/skills` in the active session; if a short name conflicts or does not resolve, ask for the workflow by name in natural language.


---

## Skills & Discovery

Antigravity automatically discovers skills inside the plugin's `skills/` directory. 
* Antigravity matches user tasks and intents to relevant skills on-demand.
* If a task matches a skill, the agent will load the skill and prompt you for permission before executing.

---

## Verification & Validation

To validate that your local plugin is correctly structured and contains all skills, run:
```bash
agy plugin validate /path/to/agent-skills
```

Then start a fresh session and inspect `/agent-skills:` in the skill catalog. Plugin validation checks structure; it does not guarantee every skill appears or that a short invocation is available in every release.

---

## How It Works

### 1. On-Demand Skill Activation
Antigravity CLI automatically discovers the `SKILL.md` files located in the `skills/` directory of the installed plugin. Using the trigger descriptions in each skill's frontmatter, the agent will dynamically activate the appropriate workflow when it detects matching developer intent.

For example, when you ask the agent to:
- **Design a new system** &rarr; It will suggest/activate `spec-driven-development`.
- **Implement a feature** &rarr; It will activate `incremental-implementation` and `test-driven-development`.
- **Fix a bug** &rarr; It will activate `debugging-and-error-recovery`.

### 2. Specialized Agent Personas
The plugin registers reusable subagent definitions from the `agents/` directory:
- `code-reviewer.md`
- `security-auditor.md`
- `test-engineer.md`

You can invoke these personas directly within your session or when delegating tasks using subagents.

---

## Configuration & Customization

### Project-Specific Enforcements (`AGENTS.md`)
To enforce strict skill compliance (e.g. requiring a spec or plan before writing code), create a project-specific `AGENTS.md` in the root of your workspace. Describe your project's conventions, test commands, and required skill workflows there.

This repository's root `AGENTS.md` and `CLAUDE.md` configure agents contributing to Agent Skills itself. Do not copy or link them into your project or global agent configuration; the reusable assets are the skills in `skills/`. See [Repo-scoped files](../CONTRIBUTING.md#repo-scoped-files).

### Sandbox Mode
If you want to run skills or scripts with limited terminal permissions (for safety when running third-party validation tests), launch the CLI with:

```bash
agy --sandbox
```

---

## Usage Tips

1. **Keep plugins up-to-date:** You can update the CLI or check for newer plugin versions using:
   ```bash
   agy update
   ```
2. **Review before execution:** When agents execute complex refactoring tasks using these skills, use `Ctrl+r` to enter the **Artifact Review** screen to review, edit, or approve code before it is committed.
3. **Control permissions:** You can use the `--dangerously-skip-permissions` flag only in trusted local projects where you want to bypass manual tool approval prompts.
