---
name: spec-workflow
description: Use only when explicitly requesting the spec-workflow skill.
---

# Specify Work

Invoke the `spec-driven-development` skill. First check whether root `SPEC.md` exists; that determines the flow.

## New specification

If there is no `SPEC.md`, or the request is unrelated to its content, understand the requested work. Ask clarifying questions about:

1. Objective and target users.
2. Core features and acceptance criteria.
3. Technology preferences and constraints.
4. Boundaries: what to always do, ask about first, and never do.

Write a structured spec covering all six areas: objective, commands, project structure, code style, testing strategy, and boundaries. If the request bundles independently testable capabilities, first propose a capability map (module IDs, dependency direction, build order) following `spec-driven-development` Phase 0; obtain approval, then specify modules in dependency order. Save as root `SPEC.md` and confirm with the user before implementation.

## Delta specification

When `SPEC.md` exists and the request changes what it describes, preserve it. Write a delta spec such as `SPEC-<change-slug>.md` covering only the change, following the Spec Evolution section of `spec-driven-development`. Confirm with the user before implementation.

## Overview
Captures requirements and boundaries in a structured specification before implementation begins.

## When to Use
- Use for a new capability or a requested change to existing specified behavior.
- Use a delta spec rather than replacing the original spec when describing a follow-up change.

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| “I can implement while clarifying the requirements.” | Confirm the spec with the user before proceeding to implementation. |
| “Several capabilities can be specified independently without a map.” | Approve module boundaries and dependency order first when capabilities are independently testable. |

## Red Flags
- Replacing an existing spec with a change-only request.
- Omitting any of the six core areas or proceeding without confirmation.

## Verification
- [ ] Correct root spec or delta spec covers all applicable core areas.
- [ ] The user has confirmed the spec before implementation.
