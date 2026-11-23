---
name: a11y-workflow
description: Use only when explicitly requesting the a11y-workflow skill.
---

# Accessibility Audit

Invoke the `accessibility-engineering` skill and spawn the `accessibility-auditor` subagent. Run an accessibility audit of the specified files, components, diff, or URL. Return the full findings; this workflow needs no synthesis.

## Select a mode

- **Deep:** Use available axe-core or pa11y JSON, a live URL, or Chrome DevTools MCP for DOM, contrast, and rendering inspection. For a URL, the user may run `bash skills/accessibility-engineering/scripts/audit-a11y.sh url <url> --json` and provide the output.
- **Quick (default):** When no deep-mode input is available, inspect source for structural issues such as missing labels, divs used as buttons, missing alt text, and skipped heading levels.

## Audit

Provide it the review scope, artifact paths or output, target URL/component if known, and expected mode. If Deep was intended but inputs are missing, surface that limitation. Return the resulting WCAG scorecard, severity-ranked findings, positive observations, and remediation guidance with code examples.

## Overview
This workflow produces an evidence-based accessibility audit with practical remediation guidance.

## When to Use
- Use for interface accessibility, WCAG 2.2 AA, keyboard, screen-reader, contrast, or form-label audits.
- Use Quick mode when there is no live target or audit artifact; use Deep mode when inspection evidence is available.

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| “Source inspection is enough for a deep audit.” | Deep findings require live or report evidence; otherwise label the source scan Quick. |

## Red Flags
- Reporting measured compliance without an audit artifact or live inspection.
- Omitting the intended mode or missing Deep-mode inputs.

## Verification
- [ ] The report identifies Quick or Deep mode and its evidence.
- [ ] Findings are ranked and include concrete remediation guidance.
