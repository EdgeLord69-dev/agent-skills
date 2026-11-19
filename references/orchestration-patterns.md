# Orchestration Patterns

Reference catalog of agent orchestration patterns this repo endorses, plus anti-patterns to avoid. Read this before changing how workflow skills compose personas or introducing a persona that "wraps" existing ones.

The governing rule: **the user is the orchestrator. Personas do not invoke other personas.** Skills can guide work within a persona or be invoked directly; they do not themselves guarantee host-specific orchestration behavior.

---

## Endorsed patterns

### 1. Direct invocation (no orchestration)

Single persona, single perspective, single artifact. The default and the cheapest option.

```
user → code-reviewer → report → user
```

**Use when:** the work is one perspective on one artifact and you can describe it in one sentence.

**Examples:**
- "Review this PR" → `code-reviewer`
- "Find security issues in `auth.ts`" → `security-auditor`
- "What tests are missing for the checkout flow?" → `test-engineer`

**Cost:** one round trip. The baseline you should always compare orchestrated patterns against.

---

### 2. Direct skill workflow

A skill captures a repeatable workflow and can be invoked by the user alongside (or independently of) a persona. Host-specific invocation syntax varies; don't assume a slash-command interface.

```
review skill → code-review-and-quality workflow → report
```

**Use when:** the work needs a repeatable process, but not necessarily a separate persona context.

**Examples in this repo:** `spec`, `plan`, `build`, `test`, `review`, `constraints`, `code-simplify`, `webperf`, `a11y`.

**Cost:** one context; the skill is workflow guidance, not another agent.

**Anti-signal:** if a skill's body is mostly "decide which persona to call," let the user select the persona directly.
---

### 3. Parallel fan-out with merge

Multiple personas operate on the same input concurrently, each producing an independent report. A merge step (in the main agent's context) synthesizes them into a single decision.

```
                    ┌─→ code-reviewer    ─┐
ship workflow → fan out ─┼─→ security-auditor ─┤→ merge → go/no-go + rollback
                    └─→ test-engineer    ─┘
```

**Use when:**
- The sub-tasks are genuinely independent (no shared mutable state, no ordering dependency)
- Each sub-agent benefits from its own context window
- The merge step is small enough to stay in the main context
- Wall-clock latency matters

**Examples in this repo:** `ship`.

**Cost:** N parallel sub-agent contexts + one merge turn. Higher than direct invocation, but faster wall-clock and produces better reports because each sub-agent stays focused on its single perspective.

**Validation checklist before adopting this pattern:**
- [ ] Can I run all sub-agents at the same time without ordering issues?
- [ ] Does each persona produce a different *kind* of finding, not just the same finding from a different angle?
- [ ] Will the merge step fit in the main agent's remaining context?
- [ ] Is the user's wait time long enough that parallelism is actually noticeable?

If any answer is "no," fall back to direct invocation or a single workflow skill.

---

### 4. Sequential pipeline as user-driven skills

The user invokes skills in a defined order, carrying context (or commit history) between them. There is no orchestrator agent — the user IS the orchestrator.

```
user runs:  spec → plan → build → test → review → ship
```

**Use when:** the workflow has dependencies (each step needs the previous step's output) and human judgment between steps adds value.

**Examples in this repo:** the entire DEFINE → PLAN → BUILD → VERIFY → REVIEW → SHIP lifecycle.

**Cost:** one user-led step at a time. No extra orchestration agent.

**Why not automate it:** an LLM "lifecycle orchestrator" would (a) lose nuance between steps because it has to summarize for hand-off, (b) skip the human checkpoints that catch wrong-direction work early, and (c) double the token cost via paraphrasing turns.

---

### 5. Research isolation (context preservation)

When a task requires reading large amounts of material that shouldn't pollute the main context, spawn a research sub-agent that returns only a digest.

```
main agent → research sub-agent (reads 50 files) → digest → main agent continues
```

**Use when:**
- The main session needs to stay focused on a downstream task
- The investigation result is much smaller than the input it consumes
- The decision quality benefits from the main agent having room to think after

**Examples:** "Find every call site of this deprecated API across the monorepo," "Summarize what these 30 ADRs say about caching."

**Cost:** one isolated sub-agent context. Worth it any time the alternative is loading hundreds of files into the main context.

**On Claude Code, use the built-in `Explore` subagent** rather than defining a custom research persona. `Explore` runs on Haiku, is denied write/edit tools, and is purpose-built for this pattern. Define a custom research subagent only when `Explore` doesn't fit (e.g. you need a domain-specific system prompt the model wouldn't infer).

---

## Host-specific implementation

The patterns above are host-agnostic. Hosts differ in how they expose skills, load persona files, and run subagents or collaborative teams; consult the host's current documentation before relying on discovery behavior, frontmatter fields, concurrency, or experimental features.

Keep orchestration semantics independent of those details:

- For parallel fan-out, the main agent starts independent persona tasks and merges their reports.
- For collaborative investigation, use a host feature that allows teammates to exchange findings, when available and appropriate.
- Never assume a persona can launch another persona, or that one host's capability or restriction applies on another.

When authoring a persona, only rely on the frontmatter fields supported by the target host. Do not claim a field is honored or ignored universally.

---

## Worked example: Agent Teams for competing-hypothesis debugging

This example shows when to use collaborative Agent Teams instead of `ship`'s subagent fan-out. The patterns both involve multiple personas, but differ in whether investigators need to exchange findings.

### The scenario

> *Checkout occasionally hangs for ~30 seconds before completing. It happens roughly once every 50 sessions. No errors in logs. Started after last week's release.*

Plausible root causes (mutually exclusive, all fit the symptoms):

1. A race condition in the new payment-confirmation flow
2. An auth check that occasionally falls through to a slow synchronous network call
3. A missing index on a query that scales with cart size
4. A flaky third-party API where the SDK retries silently before timing out

A single agent will pick the first plausible theory and stop investigating. A `ship`-style fan-out gives each persona an independent report, but the reports do not challenge one another, so nothing rules out the wrong theories.

This is exactly the case collaborative-agent-team patterns aim to address: independent investigators actively challenge one another, making the surviving theory more likely to be the actual root cause.

### Why this is *not* a `ship` job

| | `ship` (subagents) | Collaborative team |
|--|--------------------|-------------|
| Sub-agents see | The same diff, different lenses | A shared task list, each other's messages |
| Output | Three independent reports → one merge | Adversarial debate → consensus root cause |
| Right when | You want a verdict on a known artifact | You want to *find* the artifact among hypotheses |

`ship` is a verdict; a collaborative team is an investigation.
PUT 221.=231:
Three teammates running for ~10–15 minutes of investigation can cost noticeably more than the same three personas spawned as subagents by `ship`. The justification is *quality of conclusion* — for production debugging where the wrong fix is expensive, the extra tokens may be worthwhile. For a routine PR review, use `ship`.

### Anti-pattern in this scenario

Do **not** turn this into a workflow that fans out isolated subagents: they cannot challenge each other's findings, losing the adversarial debate that makes the investigation useful. If the workflow recurs, document the trigger prompt as a snippet rather than wrapping it in orchestration that cannot support collaboration.

### When *not* to use collaborative teams

- Production-bound verdict on a known diff → use `ship` (subagents).
- One specialist perspective on one artifact → direct persona invocation.
- Sequential lifecycle (spec → plan → build) → user-driven skills (Pattern 4).

### Setup (one-time, per-environment)

Agent Teams is experimental. In `~/.claude/settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Requires Claude Code v2.1.32 or later. The personas in this repo are picked up automatically — no team-config files to author by hand.

### The trigger prompt

Type into the lead session, in natural language:

```
Users report checkout hangs for ~30 seconds intermittently after last
week's release. No errors in logs.

Create an agent team to debug this with competing hypotheses. Spawn
three teammates using the existing agent types:

  - code-reviewer  — investigate race conditions and blocking calls
                     in the checkout code path
  - security-auditor — investigate auth checks, session handling,
                       and any synchronous network calls added recently
  - test-engineer  — propose tests that would distinguish between the
                     hypotheses and check coverage gaps in checkout

Have them message each other directly to challenge each other's
theories. Update findings as consensus emerges. Only converge when
two teammates agree they can disprove the others'.
```

The lead spawns three teammates referencing the existing persona names. The persona body is **appended** to each teammate's system prompt as additional instructions (on top of the team-coordination instructions the lead installs); the trigger prompt above becomes their task.

### What happens

1. Each teammate runs in its own context window, exploring the codebase from its own lens.
2. Teammates use `message` to send findings to each other directly. The lead doesn't have to relay.
3. The shared task list shows who's investigating what — visible at any time with `Ctrl+T` (in-process mode) or in a tmux pane (split mode).
4. When `code-reviewer` finds a `Promise.all` that should be sequential, it messages `security-auditor` to confirm the auth call isn't part of the race. `security-auditor` checks and replies — either confirming the race is the real issue or producing counter-evidence.
5. `test-engineer` proposes a focused integration test for whichever theory is winning, which the team uses to verify before declaring consensus.
6. The lead synthesizes the converged finding and presents it to you.

You can interrupt at any teammate by cycling with `Shift+Down` and typing — useful for redirecting an investigator who's gone down a wrong path.

### When to clean up

When the investigation lands on a root cause, tell the lead:

```
Clean up the team
```

Always cleanup through the lead, not a teammate (per the docs: teammates lack full team context for cleanup).

### Cost expectation

Three teammates running for ~10–15 minutes of investigation can cost noticeably more than the same three personas spawned as subagents by `ship`. The justification is *quality of conclusion* — for production debugging where the wrong fix is expensive, the extra tokens may be worthwhile. For a routine PR review, use `ship`.

### Anti-pattern in this scenario

Do **not** turn this into a workflow that fans out isolated subagents: they cannot challenge each other's findings, losing the adversarial debate that makes the investigation useful. If the workflow recurs, document the trigger prompt as a snippet rather than wrapping it in orchestration that cannot support collaboration.

### When *not* to use collaborative teams

- Production-bound verdict on a known diff → use `ship` (subagents).
- One specialist perspective on one artifact → direct persona invocation.
- Sequential lifecycle (spec → plan → build) → user-driven skills (Pattern 4).
- Read-heavy research with a small digest → built-in `Explore` subagent.

Reach for collaborative teams only when teammates **need** to challenge each other to produce the right answer.

Reach for Agent Teams only when teammates **need** to challenge each other to produce the right answer.

---

## Anti-patterns

### A. Router persona ("meta-orchestrator")

A persona whose job is to decide which other persona to call.

```
user request → router-persona → "this needs a review" → code-reviewer → router (paraphrases) → user
```

**Why it fails:**
- Pure routing layer with no domain value
- Adds two paraphrasing hops → information loss + roughly 2× token cost
- The user already knew they wanted a review; they can invoke the skill or persona directly
- Replicates the work that skill discovery and intent mapping in `AGENTS.md` already do

**What to do instead:** document intent → workflow mapping in `AGENTS.md`.

---

### B. Persona that calls another persona

A `code-reviewer` that internally invokes `security-auditor` when it sees auth code.

**Why it fails:**
- Personas were designed to produce a single perspective; chaining them defeats that
- The summary the calling persona passes loses context the called persona needs
- Failure modes multiply (which persona's output format wins? whose rules apply?)
- Hides cost from the user

**What to do instead:** have the calling persona *recommend* a follow-up audit in its report. The user runs the next pass.

---

### C. Sequential orchestrator that paraphrases

The user runs a sequential lifecycle such as `spec`, then `plan`, then `build`, carrying context between steps.

**Why it fails:**
- Loses the human checkpoints that catch wrong-direction work
- Each hand-off summarizes context — accumulated drift over a long pipeline
- Doubles token cost: orchestrator turn + sub-agent turn for every step
- Removes user agency at exactly the points where judgment matters most

**What to do instead:** keep the user as the orchestrator. Document the recommended sequence in `README.md` and let users invoke it.

---

### D. Deep persona trees

`ship` delegates to a coordinator that delegates to another coordinator that calls `code-reviewer`.

**Why it fails:**
- Each layer adds latency and tokens with no decision value
- Debugging becomes a multi-level investigation
- The leaf personas lose context to multiple summarization steps

**What to do instead:** keep orchestration depth at most 1 (user-invoked workflow → personas). The merge happens in the main agent.

---

## Decision flow

When considering a new orchestrated workflow, walk this flow:

```
Is the work one perspective on one artifact?
├── Yes → Direct invocation. Stop.
└── No  → Will the same composition repeat?
         ├── No  → Direct invocation, ad hoc. Stop.
         └── Yes → Are sub-tasks independent?
                  ├── No  → Sequential workflow skills run by user (Pattern 4).
                  └── Yes → Parallel fan-out with merge (Pattern 3).
                           Validate against the checklist above.
                           If any check fails → fall back to direct persona invocation or a workflow skill (Pattern 2).
```

---

## When to add a new pattern to this catalog

Add a new entry only after:

1. You've used the pattern at least twice in real work
2. You can name a concrete artifact in this repo that demonstrates it
3. You can explain why an existing pattern wouldn't have worked
4. You can describe its anti-pattern shadow (what people will mistakenly build instead)

Premature catalog entries become aspirational documentation that no one follows.
