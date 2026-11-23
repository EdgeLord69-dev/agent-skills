---
name: review-workflow
description: Use only when explicitly requesting the review-workflow skill.
---

# Code Review

Invoke the `code-review-and-quality` skill. Review current staged changes or recent commits against the spec and existing project patterns. Assess all five axes:

1. **Correctness:** Spec alignment, edge cases, and adequate tests.
2. **Readability:** Clear names, straightforward logic, and organization.
3. **Architecture:** Existing patterns, clean boundaries, and suitable abstraction.
4. **Security:** Input validation, secrets safety, and authorization; use `security-and-hardening` where appropriate.
5. **Performance:** N+1 queries and unbounded operations; use `performance-optimization` where appropriate.

Categorize findings as Critical, Required, Optional, or Nit. Return a structured review with specific file/line references and fix recommendations.

## Overview
Finds actionable risks in code changes and returns prioritized, evidence-linked recommendations.

## When to Use
- Use for staged changes or recent commits requiring a five-axis review.
- Use security and performance skills for deeper specialist analysis when warranted.

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| “The diff is small, so some review axes do not apply.” | Assess all five axes; a small change can still introduce a critical risk. |

## Red Flags
- Findings without file/line references or actionable recommendations.
- Omitting an axis or conflating severity categories.

## Verification
- [ ] All five axes were assessed and findings categorized as Critical, Required, Optional, or Nit.
- [ ] Each finding points to evidence and recommends a fix.
