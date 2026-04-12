---
spec: "{relative/path/to/spec.md}"
status: Active
date: "{YYYY-MM-DD}"
---

<!-- Budget: findings should be actionable and specific. No vague "consider security" entries. -->

# Security Review: {Title}

## Summary

{1-2 sentences: what was reviewed, which mode (spec-scoped or ad-hoc), key risk areas identified.}

**Artifacts reviewed:**
- {spec.md | plan.md | file path | directory — list what was actually analyzed}

## Findings

<!-- Each finding is a discrete, actionable item. Order by severity (critical first). -->

### 1. {Finding title}

- **Severity:** Critical | Notable | Advisory
- **Description:** {What the issue is — specific, not vague}
- **Suggestion:** {Concrete actionable recommendation — what to add, change, or specify}
- **Route:** Spec | Plan | Backlog

<!-- Add more findings as needed. If no findings, write "No security findings identified." and explain why (e.g., spec is low-risk, all boundaries are well-specified). -->

## STRIDE Coverage

<!-- Evaluate each category for relevance. Mark N/A for categories that clearly don't apply to this spec/plan. -->

| Category | Relevant? | Findings |
| :--- | :--- | :--- |
| Spoofing | Yes / No / N/A | {Finding refs or "No issues found"} |
| Tampering | Yes / No / N/A | {Finding refs or "No issues found"} |
| Repudiation | Yes / No / N/A | {Finding refs or "No issues found"} |
| Information Disclosure | Yes / No / N/A | {Finding refs or "No issues found"} |
| Denial of Service | Yes / No / N/A | {Finding refs or "No issues found"} |
| Elevation of Privilege | Yes / No / N/A | {Finding refs or "No issues found"} |

## Routing Recommendations

<!-- Summarize where findings should be routed. Only include sections with routable findings. -->

### Spec amendments
- {Finding N: suggested change to spec}

### Plan amendments
- {Finding N: suggested change to plan}

### Backlog items
- {Finding N: future hardening work}

<!-- Remove empty routing sections. If all findings are informational with no routing, replace this section with "No routing required — all findings are informational." -->
