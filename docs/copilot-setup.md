# Using agent-skills with GitHub Copilot

This guide covers Copilot in VS Code. For the standalone `copilot` command-line tool, see [copilot-cli-setup.md](copilot-cli-setup.md).

**What an install gives you:** the skills only. The short lifecycle names (`/spec`, `/plan`, `/build`, `/test`, `/review`, `/ship`, `/constraints`, `/code-simplify`, `/webperf`, and `/a11y`) are skills in this repository, not separate command wrappers. The full skill names are available too. Slash-command discovery and resolution can vary by Copilot surface and version; use `/skills` or type `/` to see the current catalog, and ask by skill name in natural language if a short name is missing or collides.

## Setup

### Copilot Instructions

Copilot supports creating agent skills using a `.github/skills`, `.claude/skills`, or `.agents/skills` directory in your repository.

```bash
mkdir -p .github/skills/test-driven-development .github/skills/code-review-and-quality

# Create files for essential skills
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .github/skills/test-driven-development/SKILL.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md > .github/skills/code-review-and-quality/SKILL.md
```

Whichever of these two routes you use, the result is one `SKILL.md` per skill directory under one of those three project paths. An installer run with a global/user flag writes somewhere else instead — check its output for the path it used. Run `/skills` in Copilot Chat to open the **Configure Skills** menu and confirm what got discovered.

For more details, refer [Creating agent skills for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-skills) and the VS Code guide to [agent skills](https://code.visualstudio.com/docs/agent-customization/agent-skills).

### Agent Personas (*.agent.md)

Copilot supports specialized agent personas. Use the agent-skills agents:

> **Important:** GitHub Copilot requires custom agent files to be named `*.agent.md`.
> Files named `*.md` are silently ignored by Copilot.
> See [VS Code custom agents docs](https://code.visualstudio.com/docs/copilot/customization/custom-agents#_custom-agent-file-structure) for details.

```bash
# Create the agents directory and copy agent definitions
mkdir -p .github/agents
cp /path/to/agent-skills/agents/code-reviewer.md .github/agents/code-reviewer.agent.md
cp /path/to/agent-skills/agents/test-engineer.md .github/agents/test-engineer.agent.md
cp /path/to/agent-skills/agents/security-auditor.md .github/agents/security-auditor.agent.md
```

Invoke agents in Copilot Chat:
- `@code-reviewer Review this PR`
- `@test-engineer Analyze test coverage for this module`
- `@security-auditor Check this endpoint for vulnerabilities`

### Custom Instructions (User Level)

For skills you want across all repositories:

1. Open VS Code → Settings → GitHub Copilot → Custom Instructions
2. Add your most-used skill summaries

## Recommended Configuration

### .github/copilot-instructions.md

GitHub Copilot supports project-level instructions via `.github/copilot-instructions.md`.

```markdown
# Project Coding Standards

## Testing
- Write tests before code (TDD)
- For bugs: write a failing test first, then fix (Prove-It pattern)
- Test hierarchy: unit > integration > e2e (use the lowest level that captures the behavior)
- Run `npm test` after every change

## Code Quality
- Review across five axes: correctness, readability, architecture, security, performance
- Every PR must pass: lint, type check, tests, build
- No secrets in code or version control

## Implementation
- Build in small, verifiable increments
- Each increment: implement → test → verify → commit
- Never mix formatting changes with behavior changes

## Boundaries
- Always: Run tests before commits, validate user input
- Ask first: Database schema changes, new dependencies
- Never: Commit secrets, remove failing tests, skip verification
```

### Specialized Agents

Use the agents for targeted review workflows in Copilot Chat.

## Lifecycle Skills

The workflows are available as skills under their short names:

| Workflow | Skill |
|----------|-------|
| Define | `spec` |
| Plan | `plan` |
| Build | `build` |
| Verify | `test` |
| Review | `review` |
| Simplify | `code-simplify` |
| Ship | `ship` |
| Constraints | `constraints` |
| Web performance | `webperf` |
| Accessibility | `a11y` |

Type `/` or run `/skills` to inspect what the current Copilot surface discovered. If a short skill name is unavailable or resolves to a host command, invoke the corresponding full skill name (for example, `spec-driven-development`) or request it in natural language. Prompt files can provide aliases in VS Code's extension-host local agents, but the Agent Host does not use prompt files; aliases are not needed for these skills.

Examples:

> Use the spec skill to write a spec for [the feature].

> Use the code-review-and-quality skill to review my staged changes.

## If skills do not appear

1. Confirm each skill is installed as its own directory containing `SKILL.md` in a supported discovery path.
2. Check the `/skills` Configure Skills menu and ensure the skill is enabled.
3. Start a fresh session after installing or changing skills.
4. Check your VS Code and Copilot Chat versions; discovery and invocation behavior can differ by host and release.

## Usage Tips

1. **Keep instructions concise** — Copilot instructions work best when focused. Summarize the key rules rather than including full skill files.
2. **Use agents for review** — The code-reviewer, test-engineer, and security-auditor agents are designed for Copilot's agent model.
3. **Reference in chat** — When working on a specific phase, paste the relevant skill content into Copilot Chat for context.
4. **Combine with PR reviews** — Set up Copilot to review PRs using the code-reviewer agent persona.
