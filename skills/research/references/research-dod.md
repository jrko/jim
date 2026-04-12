# Research Definition of Done

Self-check reference for the researcher agent and the `research:check` validation skill. Every research.md must pass all applicable items before presentation.

## Checklist

1. **Linked:** Includes relative link to source document (spec, brainstorm, debug doc) in frontmatter `spec:` field, or `"standalone"` marker for ad hoc research.

2. **Anchored:** Lists all primary file paths + line ranges for integration points. Each anchor has a 1-sentence explanation of why it's relevant.

3. **Test Template:** At least one existing test file identified with framework, setup pattern, and mock conventions. If no tests exist in the project, document that explicitly.

4. **Prior Art File Table:** Prior art entries include a file-level table (File | What It Is | Why It Matters) when the repo is accessible (best-effort).

5. **Prior Art Tiering:** Prior art with 5+ entries uses Tier 1 (Study Closely) / Tier 2 (Study for Specific Patterns) / Tier 3 (Reference Only) organization.

6. **Local-First Verified:** Phase 0 completed before any web research. If no local match exists, audit trail of Glob/Grep patterns attempted is present (e.g., `grep "auth"`, `glob **/auth/**`).

7. **No Library Sprawl:** New libraries compared against existing dependency files. Flagged when existing dependencies already cover the need.

8. **Risk-Aware:** At least one breaking change, security concern, or performance risk identified in Security & Performance section.

9. **Aligned:** Alignment statement referencing `docs/jim/VISION.md` and/or `docs/jim/ARCHITECTURE.md` present — or their absence explicitly noted.

10. **Budget:** Under 1500 words total.

11. **20-Line Rule:** No code blocks exceed 20 lines. Uses `file:line-range` + 1-sentence summary instead.

12. **No Assumptions:** Unreachable or confusing code listed under Open Questions or flagged in Peer Feedback rather than silently assumed.

13. **Peer Feedback:** If research invalidates spec requirements or plan assumptions, Peer Feedback section is present with structured signals for PM and/or Architect.

## Phase-Specific Checks

### Phase 0 — Local Archaeology
- At least one anchor identified OR explicit "no local implementation exists" with audit trail of patterns attempted.

### Phase 1 — External Intelligence
- Only triggered after Phase 0 completes.
- Skipped for bugs/refactors with no external dependencies.
- When triggered, follows WebFetch guardrails (only for referenced APIs, external examples, knowledge bases, or TODOs mentioning third-party migrations).

### Phase 2 — Alignment Validation
- Alignment statement always present, even if `docs/jim/VISION.md` and `docs/jim/ARCHITECTURE.md` are missing (note absence).

## Status Assignment

| Status | When to use |
|--------|-------------|
| `Active` | Research is complete, no upstream concerns. |
| `Needs PM Review` | Peer Feedback contains spec feasibility signals — PM should review before planning proceeds. |
| `Needs Architect Review` | Peer Feedback contains plan invalidation signals — Architect should review before implementation. |
