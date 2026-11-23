---
name: code-simplify-workflow
description: Runs the code-simplify workflow. Use when explicitly requesting the code-simplify-workflow skill.
---

# Simplify Code

Invoke the `code-simplification` skill. Preserve exact behavior while improving clarity and maintainability.

## Workflow

1. Read `CLAUDE.md` and study project conventions.
2. Identify the target code: recent changes by default, or the broader scope specified by the user.
3. Understand purpose, callers, edge cases, and test coverage before editing.
4. Look for deep nesting to replace with guard clauses or extracted helpers; long functions to split by responsibility; nested ternaries to replace with if/else or switch; vague names to clarify; duplicated logic to share; and dead code to remove only after confirming it is dead.
5. Apply simplifications incrementally and run tests after each change.
6. Verify all tests pass, the build succeeds, and the diff is clean. If a simplification fails tests, revert that change and reconsider it.
7. Review the result with `code-review-and-quality`.

## Overview
Reduces unnecessary complexity while preserving exact behavior.

## When to Use
- Use after changes or for a specified scope when clarity and maintainability can improve.
- Do not use to introduce behavior changes or redesign beyond the requested simplification.

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| “This cleanup is obviously behavior-neutral.” | Callers, edge cases, and tests must be understood, then tests must verify the change. |

## Red Flags
- Editing before understanding purpose, callers, and coverage.
- Bundling behavior changes or retaining a simplification that breaks tests.

## Verification
- [ ] Tests and build pass and the diff is clean.
- [ ] The result has been reviewed with `code-review-and-quality`.
