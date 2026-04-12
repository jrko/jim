---
spec: "{relative/path/to/spec.md}"
status: Active
date: "{YYYY-MM-DD}"
---

<!-- Budget: <1500 words total. Never paste >20 lines of code — use file:line-range + 1-sentence summary. -->

# Research: {Title}

## Anchors

<!-- Always present. File paths + line ranges for implementation AND test locations. -->
<!-- For each anchor: 1-sentence explanation of why it's an anchor. -->
<!-- Include both existing files and new files to be created. -->

## Local Patterns

<!-- Always present. Existing hooks, utilities, conventions the implementation should follow. -->
<!-- Identify at least one existing test file as template for the coder — including test framework, setup pattern, and mock conventions. -->

## Prior Art

<!-- Conditional: only for features with relevant external examples. -->
<!-- Links + synthesis: what's relevant, why, what to ignore. -->
<!-- For each entry, include a file-level table when the repo is accessible (best-effort):
| File | What It Is | Why It Matters |
|------|------------|----------------|
-->
<!-- When 5+ entries: organize into Tier 1 (Study Closely) / Tier 2 (Study for Specific Patterns) / Tier 3 (Reference Only). Under 5: flat list. -->

## Libraries

<!-- Conditional: only when new libraries are needed or refactors touch dependencies. -->
<!-- Compare against existing dependency files. Flag library sprawl. -->

## Security & Performance

<!-- Always present. Tactical risks and guardrails: auth boundaries, rate limits, n+1 queries, etc. -->

## Recommendations

<!-- Always present. Options and trade-offs for the architect (not decisions). -->
<!-- When a recommendation diverges from a locked constraint in docs/jim/VISION.md or docs/jim/ARCHITECTURE.md, explicitly note the divergence and the rationale. -->

## Peer Feedback

<!-- Conditional: only when research surfaces signals for PM or Architect. -->
<!-- For PM: spec feasibility concerns, better approaches. -->
<!-- For Architect: plan invalidation from updated research. -->
