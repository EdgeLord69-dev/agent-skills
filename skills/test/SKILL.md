---
name: test
description: Runs the test workflow. Use when explicitly requesting the test skill.
---

# Test-Driven Development

Invoke the `test-driven-development` skill for the complete workflow.

## New features

1. Write tests describing expected behavior and confirm they fail.
2. Implement code that makes them pass.
3. Refactor while keeping tests green.

## Bug fixes: Prove-It pattern

1. Write a test reproducing the bug and confirm it fails.
2. Implement the fix.
3. Confirm the test passes.
4. Run the full test suite for regressions.

For browser-related issues, also use `browser-testing-with-devtools` to verify with Chrome DevTools MCP.

## Overview
Uses failing tests to specify expected behavior, then proves the implementation and guards against regressions.

## When to Use
- Use for new features and bug fixes.
- For browser-related issues, also use `browser-testing-with-devtools`.

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| “The fix is clear, so a reproducer is unnecessary.” | A failing test proves the bug and prevents an unverified fix. |

## Red Flags
- Implementing before a test demonstrates expected behavior or reproduces the bug.
- Treating a passing focused test as proof against regressions without the full suite for bug fixes.

## Verification
- [ ] Feature tests fail before implementation and pass afterward, followed by green refactoring.
- [ ] Bug reproducer fails before the fix, passes after, and the full suite passes.
