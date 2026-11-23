# Agent Personas

Specialist personas that play a single role with a single perspective. Each persona is a Markdown file consumed as a system prompt by your harness (Claude Code, Cursor, Copilot, etc.).

| Persona | Role | Best for |
|---------|------|----------|
| [code-reviewer](../agents/code-reviewer.md) | Senior Staff Engineer | Five-axis review before merge |
| [security-auditor](../agents/security-auditor.md) | Security Engineer | Vulnerability detection, OWASP-style audit |
| [test-engineer](../agents/test-engineer.md) | QA Engineer | Test strategy, coverage analysis, Prove-It pattern |
| [web-performance-auditor](../agents/web-performance-auditor.md) | Web Performance Engineer | Core Web Vitals audit, loading/rendering/network analysis |
| [accessibility-auditor](../agents/accessibility-auditor.md) | Accessibility Engineer | WCAG 2.2 AA conformance audit, keyboard and screen reader review, contrast |

## How personas relate to skills

Two layers have distinct jobs:

| Layer | What it is | Example | Composition role |
|-------|-----------|---------|------------------|
| **Skill** | A workflow with steps and exit criteria | `code-review-and-quality` | The *how* — invoked from inside a persona or by the user |
| **Persona** | A role with a perspective and an output format | `code-reviewer` | The *who* — adopts a viewpoint and produces a report |

The user is the orchestrator. **Personas do not call other personas.** Skills can guide work inside a persona or be invoked directly by the user; they are not persona wrappers or a separate orchestration layer.

## Workflow skills

Use the relevant skill directly for a repeatable workflow:

- `spec-workflow` → define requirements and scope
- `plan-workflow` → break work into an actionable plan
- `build-workflow` → implement planned changes
- `test-workflow` → develop and verify tests
- `review-workflow` → review changes
- `ship-workflow` → coordinate review, security, and test perspectives, then synthesize a go/no-go decision
- `constraints-workflow` → identify and apply quality-bar constraints
- `code-simplify-workflow` → simplify working code without changing behavior
- `webperf-workflow` → audit web performance
- `a11y-workflow` → audit accessibility

Skills are discoverable by the harness; invocation syntax varies by host. Use the skill by name or ask naturally for its workflow rather than assuming every host exposes slash commands.

The workflow skills do not replace the specialist personas. For a focused report in a fresh context, invoke the relevant persona directly; `ship-workflow` is the workflow that composes `code-reviewer`, `security-auditor`, and `test-engineer` in parallel and merges their reports.

## When to use each

### Direct persona invocation
Pick this when you want one perspective on the current change and the user is in the loop.

- "Review this PR" → invoke `code-reviewer` directly
- "Are there security issues in `auth.ts`?" → invoke `security-auditor` directly
- "What tests are missing for the checkout flow?" → invoke `test-engineer` directly
- "Audit Core Web Vitals on the product page" → invoke `web-performance-auditor` directly
- "Is this page usable with a screen reader?" → invoke `accessibility-auditor` directly

### Parallel fan-out with merge
Use `ship-workflow` only when independent investigations can run in parallel and produce reports that a single agent then merges:

- `ship-workflow` → fans out to `code-reviewer` + `security-auditor` + `test-engineer` in parallel, then synthesizes their reports into a go/no-go decision

This is the only multi-persona orchestration pattern this repo endorses. See [references/orchestration-patterns.md](../references/orchestration-patterns.md) for the full pattern catalog and anti-patterns.

## Decision matrix

```
Is the work a single perspective on a single artifact?
├── Yes → Direct persona invocation
└── No  → Are the sub-tasks independent (no shared mutable state, no ordering)?
         ├── Yes → Use ship-workflow for parallel fan-out and merge
         └── No  → User runs workflow skills in sequence (spec-workflow → plan-workflow → build-workflow → test-workflow → review-workflow → ship-workflow)

## Worked example: valid orchestration

`ship-workflow` is the canonical fan-out workflow in this repo:

```
ship-workflow
  ├── (parallel) code-reviewer    → review report
  ├── (parallel) security-auditor → audit report
  └── (parallel) test-engineer    → coverage report
                  ↓
        merge phase (main agent)
                  ↓
        go/no-go decision + rollback plan
```

Why this works:
- Each sub-agent operates on the same diff but produces a **different perspective**
- They have no dependencies on each other → genuine parallelism, real wall-clock savings
- Each runs in a fresh context window → main session stays uncluttered
- The merge step is small and benefits from full context, so it stays in the main agent

## Worked example: invalid orchestration (do not build this)

A `meta-orchestrator` persona whose job is "decide which other persona to call":

```
user request → meta-orchestrator
                  ↓ (decides "this needs a review")
              code-reviewer
                  ↓ (returns)
              meta-orchestrator (paraphrases result)
                  ↓
                  user
```

Why this fails:
- Pure routing layer with no domain value
- Adds two paraphrasing hops → information loss + 2× token cost
- The user already knows they want a review; let them invoke the workflow skill or persona
- Replicates work that skill discovery and `AGENTS.md` intent-mapping already do

## Rules for personas

1. A persona is a single role with a single output format. If you find yourself adding a second role, create a second persona.
2. **Personas do not invoke other personas.** Composition is the job of the user or a workflow such as `ship-workflow`.
3. A persona may invoke skills (the *how*).
4. Every persona file ends with a "Composition" block stating where it fits.

## Claude Code interop

The personas in this repo can be used as Claude Code subagents and Agent Teams teammates, subject to the host's supported configuration:

- **As subagents:** use the Agent tool with the persona name (for example, `code-reviewer`) when available.
- **As Agent Teams teammates:** where supported and enabled, reference the same persona name when spawning a teammate. Agent Teams are experimental and availability depends on the installed Claude Code version.

Subagents report results back to the main agent. Agent Teams can allow teammates to message each other directly. Use subagents when reports are enough; use Agent Teams when teammates need to challenge each other's findings. See [references/orchestration-patterns.md](../references/orchestration-patterns.md).

Do not assume platform-specific discovery, frontmatter support, or experimental feature availability across hosts; consult the host's current documentation.

## Adding a new persona

1. Create `agents/<role>.md` with the same frontmatter format used by existing personas.
2. Define the role, scope, output format, and rules.
3. Add a **Composition** block at the bottom.
4. Add the persona to the table at the top of this file.
5. If the persona enables a new orchestration pattern, document it in `references/orchestration-patterns.md` rather than inventing the pattern in the persona file itself.

