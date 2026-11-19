---
name: build
description: Executes the build workflow for planned tasks, including auto mode. Use when explicitly requesting the build skill.
---

# Build Planned Work

`$ARGUMENTS` selects mode: treat `auto` (canonical) or `all` as autonomous execution; any other value or empty arguments means implement one next task and stop. Also recognize natural-language intent such as “build all,” “run the whole plan,” or “do every task” as autonomous mode even when a host passes the full phrase rather than a bare token. Both modes invoke `incremental-implementation` alongside `test-driven-development`. Autonomous mode removes pauses between tasks, not verification.

## One task

Pick the next pending plan task. Read its acceptance criteria and relevant code, patterns, and types. Write a failing test (RED), implement the minimum to pass (GREEN), run the full test suite and build, commit descriptively, mark the task complete, and stop.

## Autonomous plan

1. Require a spec at `SPEC.md`, a root `SPEC-<change>.md`, `docs/SPEC.md`, or under `spec/`. For follow-up work, a matching delta spec takes precedence over `SPEC.md`; among deltas select one whose `Status:` is not `implemented`. If multiple remain, ask which to use. A README or arbitrary document is not a spec. If none exists, stop and tell the user to run the spec workflow; do not invent requirements.
2. Check `git status --porcelain`. If uncommitted changes exist outside expected planning artifacts (`SPEC.md`, root `SPEC-*.md`, `docs/SPEC.md`, `spec/*`, `tasks/plan.md`, `tasks/todo.md`), stop and ask the user to commit, stash, or confirm handling. Do not let task commits absorb unrelated work.
3. If `tasks/plan.md` is absent, use planning-and-task-breakdown to create it and add `Source spec: <path>` at the top. If it exists, reuse only if its `Source spec:` matches the selected spec. On mismatch, including no source line, stop and ask whether to archive or replace it.
4. Present the full plan and wait for unambiguous approval (for example, “approve”, “go”, or “yes”). Hedged approval is not approval. This is the sole human checkpoint. Commit a newly generated plan as a preparatory commit before task commits.
5. Execute tasks in dependency order, or listed order when dependencies are unspecified. For each, use the full one-task loop. Stage only files touched by that task plus its status update; make one commit per task for clean rollback.
6. Stop for user input if tests cannot pass or build fails without an obvious fix (use debugging-and-error-recovery), the spec leaves a decision open, or work is high-risk/irreversible (auth, permissions, destructive migrations, payments, deletion, deploys, secrets, or anything not undoable with `git revert`). For high-risk work, use doubt-driven-development and obtain explicit sign-off. After resolution, the user re-invokes this workflow to resume.
7. Summarize completed tasks, tests added, commits, and anything skipped, flagged, or left to the user.

If any step fails, follow debugging-and-error-recovery.

## Overview
Implements planned work with test-first verification, clean task commits, and either a single-task or approved whole-plan mode.

## When to Use
- Use to execute an existing implementation plan one task at a time.
- Use `auto`, `all`, or clear natural-language whole-plan intent to run all approved tasks.

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| “The spec is obvious, so I can start without one.” | Autonomous execution requires an approved, discoverable spec and a plan tied to it. |
| “One commit for everything is simpler.” | Per-task commits preserve clean rollback and prevent unrelated changes from being absorbed. |

## Red Flags
- Starting autonomous work without approval, a matching plan, or a clean baseline.
- Skipping the failing-test step, verification, or task-scoped staging.

## Verification
- [ ] Each completed task passed its test/build loop and has its own scoped commit.
- [ ] Task status and final summary accurately describe completed and blocked work.
