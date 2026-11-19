---
name: ship
description: Runs the ship readiness workflow and specialist fan-out. Use when explicitly requesting the ship skill.
---

# Ship Readiness

Invoke the `shipping-and-launch` skill. Assess the current change and produce one go/no-go decision with a rollback plan.

## Phase A: Independent specialist reviews

Spawn these exact specialist subagents concurrently when parallel-agent support is available:

1. **`code-reviewer`:** Run a five-axis review (correctness, readability, architecture, security, performance) on staged changes or recent commits. Return the standard review template.
2. **`security-auditor`:** Run a vulnerability and threat-model review, including OWASP Top 10, secrets handling, auth/authz, and dependency CVEs. Return the standard audit report.
3. **`test-engineer`:** Analyze coverage gaps in happy paths, edge cases, error paths, and concurrency. Return the standard coverage analysis.

When the harness lacks parallel-agent support, invoke each persona's system prompt sequentially and merge the reports. Personas work independently and do not delegate. User-defined persona instructions take precedence when available.

Skip fan-out only when all are true: at most two files, diff under 50 lines, and no auth, payments, data access, or config/environment changes. Otherwise fan out by default.

## Phase B: Synthesize

After all reports return, synthesize in the main context:

- Code quality: aggregate Critical/Required findings and failing test, lint, or build output; resolve duplicates.
- Security: Critical/High audit findings are launch blockers; cross-check review security findings.
- Performance: include review performance findings and Core Web Vitals where applicable.
- Accessibility: verify keyboard navigation, screen-reader support, and contrast directly or use an accessibility checklist.
- Infrastructure: verify environment variables, migrations, monitoring, and feature flags.
- Documentation: verify README, ADRs, and changelog.

## Phase C: Decide and report

Return the complete specialist reports and this decision structure:

```markdown
## Ship Decision: GO | NO-GO

### Blockers (must fix before ship)
- [Source persona: Critical finding + file:line]

### Recommended fixes (should fix before ship)
- [Source persona: Important finding + file:line]

### Acknowledged risks (shipping anyway)
- [Risk + mitigation]

### Rollback plan
- Trigger conditions: [signals that prompt rollback]
- Rollback procedure: [exact steps]
- Recovery time objective: [target]

### Specialist reports (full)
- [code-reviewer report]
- [security-auditor report]
- [test-engineer report]
```

A rollback plan is mandatory for GO. Any Critical finding defaults to NO-GO unless the user explicitly accepts the risk.

## Overview
Combines independent specialist review with direct launch-readiness checks into a go/no-go decision and rollback plan.

## When to Use
- Use when evaluating a production-bound change for release.
- Skip specialist fan-out only when every stated small-diff exclusion criterion is satisfied.

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| “The change looks small, so parallel review is unnecessary.” | Skip only if file count, diff size, and sensitive-area criteria all qualify. |
| “A GO decision can wait for a rollback plan.” | A rollback plan is mandatory before GO. |

## Red Flags
- Missing specialist report, synthesis, or rollback details.
- A Critical finding with a GO verdict absent explicit user risk acceptance.

## Verification
- [ ] Required personas ran independently and findings were synthesized across all readiness areas.
- [ ] Decision includes blockers, risks, full reports, and an actionable rollback plan.
