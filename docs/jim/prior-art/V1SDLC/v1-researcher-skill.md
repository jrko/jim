---
name: v1-researcher-skill
description: Tactical patterns for codebase archaeology, research, and validation.
---

# V1 Researcher Skill

## 1. Discovery Tactics

### Pattern Matching
To identify existing conventions, search for:
- **Middleware:** Grep for middleware registration patterns (e.g., guards, route middleware, app-level hooks).
- **Data Layer:** Find where the DB client is initialized and how queries are structured.
- **Styles/UI:** Search for theme configuration or the global CSS entry point.

### The "Umbrella" Search
When investigating a spec in a group (e.g., `auth`):
1. **List all increments:** `Glob` for `docs/specs/{group}/*/spec.md`
2. **Identify the baseline:** Read the most recent `approved` or `complete` spec in that folder to understand the current technical state.
3. **Find anchors:** Identify 2-3 specific file paths where the new spec must integrate.

## 2. Web Research Guardrails
Use `WebFetch` only when:
- The spec references examples of a similar features already implemented to study for prior art.
- The spec references an external API/webhook/example/knowledge base.
- Code contains TODOs mentioning a 3rd-party migration.

## 3. Definition of Done
Before stopping and asking for human review, ensure `research.md` meets these criteria:
- [ ] **Linked:** Includes relative link to source `spec.md`.
- [ ] **Anchored:** Lists all primary file paths + line ranges for integration points (highlight the 2-3 most critical). Include important API & library integration points and provide calling signature (ex. function signature, class/method signature, api call signature)
- [ ] **References:** Lists all documentation references, external codebases/github repos used to collect needed knowledge to create the plan and implementation for the source `spec.md`.
- [ ] **Reference Summary:** Summarizes all relevant references to extract and synthesize the most important details needed to construct the plan and implementation for the source `spec.md`
- [ ] **Prior Art:** Lists all prior art, (ex. external codebases/github repos; blog posts; etc.)
- [ ] **Prior Art Summary:** Summarizes all prior art to extract and synthesize the most important details needed to construct the plan and implementation for the source `spec.md`
- [ ] **Risk-Aware:** Identifies at least one breaking change or tech debt item.
- [ ] **No Assumptions:** Unreachable/confusing code listed under "Open Questions."
- [ ] **No Dead Code:** Flagged any spec targets with no imports/exports.
- [ ] **No Library Sprawl:** Flagged new libraries when existing ones suffice.

## 4. Token Efficiency
- **20-Line Rule:** Never paste >20 lines. Use `file:line-range` + 1-sentence summary.
- **Link, Don't Paste:** For 3rd-party docs, provide URL + 3 key constraints.
- **Reference by ID:** Cite prior specs by ID (e.g., "See 001-setup") instead of re-explaining.
- **Budget:** Aim for <1500 words in `research.md`. Summarize low-risk areas; detail only Risks, References, and Anchors.
