# Getting Started with agent-skills

agent-skills works with any AI coding agent that accepts Markdown instructions. This guide covers the universal approach. For tool-specific setup, see the dedicated guides.

Want a worked example before setting up your own project? The
[interactive tutorials](https://skills.addy.ie/tutorials/) walk through a
greenfield build, a brownfield feature, and a safe automation loop with
copyable prompts for Claude Code, Codex, or any other agent.

## How Skills Work

Each skill is a Markdown file (`SKILL.md`) that describes a specific engineering workflow. When loaded into an agent's context, the agent follows the workflow — including verification steps, anti-patterns to avoid, and exit criteria.

**Skills are not reference docs.** They're step-by-step processes the agent follows.

## Quick Start (Any Agent)

### 1. Clone the repository

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

### 2. Choose a skill

Browse the `skills/` directory. Each subdirectory contains a `SKILL.md` with:
- **When to use** — triggers that indicate this skill applies
- **Process** — step-by-step workflow
- **Verification** — how to confirm the work is done
- **Common rationalizations** — excuses the agent might use to skip steps
- **Red flags** — signs the skill is being violated

### 3. Use the skill

Ask your agent to follow the relevant workflow by its natural-language name, such as: "Follow the test-driven-development process for this change."

### 4. Use the meta-skill for discovery when needed

If your agent does not route skills natively, ask it to use the `using-agent-skills` workflow. It contains a flowchart that maps task types to the appropriate skill.

If your host already discovers and activates skills from their descriptions, let it activate skills on demand instead of also loading `using-agent-skills` into an always-on system prompt or rules file. That creates two routers for the same task.

### Existing projects need no migration

Install the pack from the existing project's root using the normal setup path
for your agent, then keep working in that project. Skills activate for matching
tasks; they do not require a new repository layout or a one-time conversion of
existing code.

Do not copy this repository's root `AGENTS.md` or `CLAUDE.md` into the project.
Those files configure contributors to agent-skills itself. Add only the skills
and any project-specific instructions your agent normally reads. For a gradual
rollout in an established codebase, follow the [Adoption Guide](adoption-guide.md).

## Recommended Setup

Rolling out to a real project? The **[Adoption Guide](adoption-guide.md)** covers two end-to-end paths: the full lifecycle from day one for a greenfield project, and an incremental, verification-first rollout for an established codebase. The setup below is the quick version.

### Minimal (Start here)

Load three essential skills into your rules file:

1. **spec-driven-development** — For defining what to build
2. **test-driven-development** — For proving it works
3. **code-review-and-quality** — For verifying quality before merge

These three cover the most critical quality gaps in AI-assisted development.

### Full Lifecycle

For comprehensive coverage, load skills by phase:

```
Starting a project:  spec-driven-development → planning-and-task-breakdown
During development:  incremental-implementation + test-driven-development
Before merge:        code-review-and-quality + security-and-hardening
Before deploy:       shipping-and-launch
```

### Context-Aware Loading

Don't load all skills at once — it wastes context. Load skills relevant to the current task:

- Working on UI? Load `frontend-ui-engineering`
- Debugging? Load `debugging-and-error-recovery`
- Setting up CI? Load `ci-cd-and-automation`

## Skill Anatomy

Every skill follows the same structure:

```
YAML frontmatter (name, description)
├── Overview — What this skill does
├── When to Use — Triggers and conditions
├── Core Process — Step-by-step workflow
├── Examples — Code samples and patterns
├── Common Rationalizations — Excuses and rebuttals
├── Red Flags — Signs the skill is being violated
└── Verification — Exit criteria checklist
```

See [skill-anatomy.md](skill-anatomy.md) for the full specification.

## Using Agents

The `agents/` directory contains pre-configured agent personas:

| Agent | Purpose |
|-------|---------|
| `code-reviewer.md` | Five-axis code review |
| `test-engineer.md` | Test strategy and writing |
| `security-auditor.md` | Vulnerability detection |
| `web-performance-auditor.md` | Core Web Vitals & performance audit |
| `accessibility-auditor.md` | WCAG 2.2 AA conformance audit |

Load an agent definition when you need specialized review. For example, ask your coding agent to "review this change using the code-reviewer agent persona" and provide the agent definition.

## Using workflow skills

The short-named skills under `skills/` compose the specialist workflows. Ask your agent to follow a workflow by name:

| Workflow | Specialist skills |
|---------|---------------|
| Spec-driven development | spec-driven-development |
| Constraint-driven development | constraint-driven-development |
| Planning and task breakdown | planning-and-task-breakdown |
| Incremental implementation | incremental-implementation + test-driven-development |
| Automatic implementation after plan approval | planning-and-task-breakdown → incremental-implementation + test-driven-development |
| Test-driven development | test-driven-development |
| Code review and quality | code-review-and-quality |
| Code simplification | code-simplification |
| Shipping and launch | shipping-and-launch |
| Web performance audit | web-performance-auditor (specialist agent, web apps only) |
| Accessibility audit | accessibility-engineering + accessibility-auditor (specialist audit, interfaces only) |


## Using References

The `references/` directory contains supplementary checklists:

| Reference | Use With |
|-----------|----------|
| `testing-patterns.md` | test-driven-development |
| `performance-checklist.md` | performance-optimization |
| `security-checklist.md` | security-and-hardening |
| `accessibility-checklist.md` | frontend-ui-engineering |
| `definition-of-done.md` | all skills / every change |
| `observability-checklist.md` | observability-and-instrumentation |
| `orchestration-patterns.md` | doubt-driven-development |

Load a reference when you need detailed patterns beyond what the skill covers.

If you install one skill with `npx skills add ... --skill <name>`, only the
selected `skills/<name>/` directory is copied. The skill still works, but paths
to supplementary checklists in the repo-level `references/` directory are
unavailable. Use a whole-repo integration, clone the repository, or copy the
needed checklist into a `references/` directory inside the installed skill.
This portability gap is tracked in
[addyosmani/agent-skills#361](https://github.com/addyosmani/agent-skills/issues/361).

## Spec and task artifacts

The specification and planning workflows create working artifacts (`SPEC.md`, `tasks/plan.md`, `tasks/todo.md`). Treat them as **living documents** while the work is in progress:

- Keep them in version control during development so the human and the agent have a shared source of truth.
- Update them when scope or decisions change.
- If your repo doesn’t want these files long‑term, delete them before merge or add the folder to `.gitignore` — the workflow doesn’t require them to be permanent.

### Working across sessions

The same artifacts are the handoff between sessions. For a small task, run the whole lifecycle in one session. For anything non-trivial, a fresh session per phase (spec → plan → build → review) keeps context focused — what carries the work forward is the approved files, not the conversation:

- the spec — `SPEC.md`, or wherever your spec actually lives
- `tasks/plan.md` and `tasks/todo.md` — or the external tracker the plan identifies, if you use one

**Before switching**, make sure those files reflect the decisions that still apply, the scope you approved, the questions still open, the next task, and the current verification state (which tests ran, against what).

**In the new session**, read the actual files and look at `git status` before doing anything. Don't assume approvals you can't see in the artifacts. Treat a recorded "tests pass" as a claim about a specific baseline: re-run the checks it covers if the code has moved since, if it doesn't say what was run against what, or if you're about to touch the area it covered. If the baseline still holds, take it and get on with the next task — the point is a check proportional to what changed, not a full suite at every handoff.

#### Task-boundary restarts and Ralph loops

The build workflow's automatic mode can run the whole approved plan in one session. It does not require or perform a fresh process per task. Its per-task status updates, verification results, and commits make each completed task a restartable boundary, so a capable external harness may exit and resume there without depending on chat history.

A shell-level "Ralph loop" is harness behavior, not a separate skill workflow. If you use one, restart only after the current task has reached a recorded boundary; on re-entry, read the durable artifacts and repository state before selecting the next pending task. A process exit is not evidence that a task passed, and a restart must not bypass an approval gate. See the `context-engineering` skill's **Restartable Session Boundaries** section for the handoff checklist.

Plain requests work in any agent, including a `npx skills add` install that only has the skills:

> Read SPEC.md, then break it into small verifiable tasks with acceptance criteria and dependency order. Save them to tasks/plan.md and tasks/todo.md. No product code yet — show me the plan first.

> Read SPEC.md, tasks/plan.md and tasks/todo.md, then check where things actually stand — `git status`, plus re-running whatever checks the recorded verification state no longer covers. Tell me the next unchecked task and anything still open, then stop: I'll confirm the scope before you start it. If the plan looks incomplete, say what's missing rather than rewriting it.

## Tips

1. **Start with spec-driven-development** for any non-trivial work
2. **Always load test-driven-development** when writing code
3. **Don't skip verification steps** — they're the whole point
4. **Load skills selectively** — more context isn't always better
5. **Use the agents for review** — different perspectives catch different issues
