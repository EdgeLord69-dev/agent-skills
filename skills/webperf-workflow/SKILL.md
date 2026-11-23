---
name: webperf-workflow
description: Runs the webperf auditor workflow. Use when explicitly requesting the webperf-workflow skill.
---

# Web Performance Audit

This workflow is for web applications, not utility libraries, CLIs, or server-only code without browser-facing output.

## Select a mode

- **Deep:** Use available Lighthouse JSON (for example, `npx lighthouse <url> --output json --output-path ./report.json` or Chrome DevTools MCP CLI), PageSpeed Insights JSON, CrUX API response (requires `CRUX_API_KEY` or `GOOGLE_API_KEY`), a DevTools performance trace, or a live URL with Chrome DevTools MCP configured. The MCP CLI can capture via `lighthouse_audit` and `performance_*` tools; the user may run `chrome-devtools lighthouse_audit --output-format=json` and provide its JSON.
- **Quick (default):** Without deep-mode evidence, inspect source for structural anti-patterns and label every finding `potential impact`.

## Audit

Spawn the `web-performance-auditor` subagent. Provide the files/components/diff, artifact paths or pasted JSON, target URL/page if known, and expected mode. If Deep mode was intended but inputs are missing, surface that limitation. Return the scorecard populated only by sourced values, ranked findings, positive observations, and proactive recommendations. Return the full report without an additional synthesis step.

## Overview
Produces a sourced performance scorecard and prioritized recommendations for a browser-facing web application.

## When to Use
- Use for browser-rendered web applications and pages.
- Do not use for utility libraries, CLIs, or server-only code without browser output.

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| “A source scan provides measured performance scores.” | Quick findings are potential impacts; scores require sourced audit data. |

## Red Flags
- Presenting unsourced values as measured scores.
- Omitting `potential impact` labels for Quick-mode findings or hiding missing Deep inputs.

## Verification
- [ ] Audit mode and supporting evidence are clear; every score is sourced.
- [ ] Findings are ranked, positives and recommendations included, and the full report returned.
