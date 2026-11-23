---
name: constraints-workflow
description: Runs the constraints setup, check, guard, or ratchet workflow. Use when explicitly requesting the constraints-workflow skill.
---

# Repository Constraints

Invoke the `constraint-driven-development` skill. `$ARGUMENTS` selects an operation. With no arguments, set up repository constraints.

## Setup

1. **Detect:** Inspect `package.json` / `pyproject.toml` / `go.mod`, test runner, lint config, current coverage output, CI workflows, and agent harness. Report findings in two lines; do not ask for facts available in the repository.
2. **Interview:** Ask at most four questions, one at a time, each with a best guess and usable default: quality dimensions beyond the floor (coverage, security, performance, accessibility, architecture); block or warn on mid-task failures; target numbers or measure and hold current values; and slowest acceptable check before returning work.
3. Write root `CONSTRAINTS.md` with a Floor, enforced numbers, measured-only metrics and current values, and an exceptions table with owners and expiry dates. State a reason for every number.
4. Install tooling for each selected dimension; a number without an enforcing tool is only an aspiration. Prefer tools that preserve existing config: Semgrep (code scanning), gitleaks with `--redact` (secrets), osv-scanner (dependencies), axe-core (accessibility), Lighthouse (web vitals), size-limit (bundles), dependency-cruiser (boundaries), Stryker (assertion quality). Record each exact command beside its rule. Accessibility and performance require a running URL; if none exists, explain and omit that dimension rather than inventing a check. Add commands as `check:fast`, `check:task`, and `check:full` in `package.json`.
5. Place types, lint, and secrets checks in the seconds-long edit loop; related tests and changed-line coverage at task end (under 90 seconds); other checks at review or CI. Scope checks to the diff.
6. Add a line to `CLAUDE.md` instructing agents to read `CONSTRAINTS.md` and never weaken it to pass a change.
7. Run the constraints on the current branch. If the user disagrees with a failing constraint, correct the constraint now rather than leaving a gate that will be ignored.

## Operations

- `check`: run current constraints on this branch and report results.
- `guard`: inspect the diff for a weakened bar: lowered thresholds, skipped/deleted tests, new suppressions, unfinished stubs, or new exceptions.
- `ratchet`: record today's measured values as the floor that must not fall.

## Overview
Establishes measurable, enforceable repository quality constraints and provides operations to check, guard, or ratchet them.

## When to Use
- Use without arguments to establish the repository quality bar.
- Use `check`, `guard`, or `ratchet` to apply existing constraints.

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| “A threshold documents intent even without tooling.” | A number without an enforcing check is only an aspiration. |
| “A failed gate can stay as-is even if nobody accepts it.” | Correct disputed constraints now so teams do not learn to ignore the gate. |

## Red Flags
- Numbers without reasons, commands, or named exceptions.
- Accessibility or performance gates with no running URL.

## Verification
- [ ] `CONSTRAINTS.md` defines justified floors, metrics, commands, and owned/dated exceptions.
- [ ] Checks ran against the current branch and disputed failures were resolved.
