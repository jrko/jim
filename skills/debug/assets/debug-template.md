---
title: "{topic}"
date: "{YYYYMMDD}"
spec: "{path/to/related/spec.md or 'none'}"
plan: "{path/to/related/plan.md or 'none'}"
---

# Debug Report: {topic}

## Error Analysis

*What failed. Paste or describe the error output, exception, or unexpected behavior. Include: error type, message, file, line number if available.*

```
{error output here}
```

**Observed behavior:** {what actually happened}

**Expected behavior:** {what should have happened}

---

## Reproduction Steps

*Exact steps to reproduce the failure. Be specific enough that another developer can reproduce it from scratch.*

1. {step 1}
2. {step 2}
3. ...

**Reproduction command (if applicable):**
```
{command to reproduce}
```

**Can reproduce:** {yes / no / intermittent}

---

## Root Cause Hypothesis

*What is causing the failure and why. Distinguish between confirmed facts (evidence from code/logs) and hypotheses (inferred causes).*

**Most likely cause:** {description}

**Evidence:**
- {file:line or log entry that supports this}
- ...

**Alternative hypotheses:**
- {other possible cause, if any}

**Affected code locations:**
- `{file path}:{line range}` — {why this is relevant}

---

## Affected Specs/Plans

*Link any spec or plan that is relevant to this failure. This enables the `origin:` field in bug specs to reference this report.*

| Artifact | Path | Relationship |
| :--- | :--- | :--- |
| Spec | {path or 'none'} | {e.g., "defines the behavior that is broken"} |
| Plan | {path or 'none'} | {e.g., "task 3 is the likely source"} |

---

## Recommended Next Step

*One clear recommendation. Do not list multiple options — pick the most appropriate given the diagnosis.*

**Recommendation:** {one of the options below}

**Option A — Direct fix:** The root cause is clear and bounded. The fix is a small, targeted change with no spec implications. Implement via the normal TDD loop (Red: write reproduction test, Green: fix, commit).

**Option B — Plan update needed:** The root cause requires a change to how a plan task is structured. Bring the diagnosis to `@jim:architect` with the task number and the specific change needed.

**Option C — Spec update needed:** The diagnosis reveals a fundamental flaw or gap in the original requirements — the code is doing what the spec says, but the spec is wrong or incomplete. Use `/jim:spec` to open a new spec (bug type) that captures the correct behavior. This report should be referenced in the new spec's `origin:` field.

**Chosen recommendation:** {A / B / C} — {one sentence rationale}
