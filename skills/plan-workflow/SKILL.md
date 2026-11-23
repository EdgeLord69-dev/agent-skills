---
name: plan-workflow
description: Use only when explicitly requesting the plan-workflow skill.
---

# Plan Work

Invoke the `planning-and-task-breakdown` skill. Read the existing spec (`SPEC.md` or equivalent) and relevant codebase sections. Stay read-only while planning.

1. Enter plan mode: make no code changes.
2. Identify the dependency graph between components.
3. Slice work vertically, with one complete path per task rather than horizontal layers.
4. Give every task acceptance criteria and verification steps.
5. Add checkpoints between phases.
6. Present the plan for human review.
7. Save the plan to `tasks/plan.md` and task list to `tasks/todo.md`.

If either target file already contains unchecked tasks for different work, stop and ask before writing. Never silently overwrite an incomplete plan.

## Overview
Produces a reviewable, dependency-aware plan whose tasks each describe a complete vertical slice and its proof of completion.

## When to Use
- Use to break specified work into implementation tasks and verification checkpoints.
- Keep this workflow read-only until the plan is presented and saved.

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| “The existing unchecked plan is probably obsolete.” | Confirm with the user before overwriting unfinished work for a different task. |

## Red Flags
- Horizontal layer-only tasks with no complete user-visible path.
- Tasks missing acceptance criteria or verification steps.

## Verification
- [ ] Plan and task list are saved at the requested paths without overwriting unrelated unchecked work.
- [ ] Dependencies, acceptance criteria, verification, and phase checkpoints are explicit.
