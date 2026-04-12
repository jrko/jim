---
spec: "docs/jim/specs/010-sec/spec.md"
status: Active
date: "2026-04-11"
---

# Research: Security Agent and Skill

## Anchors

- **`agents/pm.md`** (L1–75) — Agent persona template. PM is the closest analogue: reads strategic docs, produces artifacts, routes findings. Security agent follows this pattern but with a security lens.
- **`agents/architect.md`** (L1–81) — Demonstrates `Agent(researcher)` spawn syntax and how an analytical agent structures its process.
- **`skills/spec/SKILL.md`** (L1–190) — Most complete skill example. Shows argument routing table, process steps, validation checklist, differential update mode, and asset/reference split.
- **`skills/debug/SKILL.md`** (L1–100) — Analytical skill that reviews artifacts and produces findings. Closest process analogue to `/jim:sec`.
- **`skills/research/SKILL.md`** (L1–120) — Shows how a skill adapts behavior based on input type (spec path vs directory vs topic string). Model for `/jim:sec` mode detection (spec-scoped vs ad-hoc).
- **`docs/jim/ARCHITECTURE.md`** (L191–197) — Existing security considerations section. The security agent should read this to avoid contradicting established trust boundaries.
- **`skills/plan/references/plan-dod.md`** (L50–54) — Existing security boundary checks in plan validation. `/jim:sec` should complement these, not duplicate them.

## Local Patterns

**Agent frontmatter convention** — All agents use: `name`, `description` (with examples), `skills`, `tools`, `model: sonnet`. Body is second-person imperative, ≤800 tokens. The security agent follows this exactly.

**Tool permissions by role:**
- Read-only analysts: `[Read, Glob, Grep, Write, Edit, Agent(Explore)]` (researcher)
- Artifact producers with spawn: `[Read, Write, Edit, Glob, Grep, Agent(researcher)]` (pm, architect)
- The security agent needs: `[Read, Write, Edit, Glob, Grep, Agent(researcher)]` — it produces `security.md` and may spawn researcher for deeper investigation.

**Skill structure convention** — Frontmatter (`name`, `description`, `agent`, `argument-hint`) + argument routing table + numbered process steps + validation checklist. Assets in `assets/`, methodology in `references/`.

**Differential update pattern** — Universal across all artifact-producing skills: read existing, summarize changes, ask user, use Edit not Write. `/jim:sec` follows this for re-runs.

**Sibling artifact pattern** — Spec directories contain `spec.md`, `research.md`, `plan.md`. Adding `security.md` extends the pattern naturally. No structural changes needed.

**Test template:** No automated test suite exists — jim is pure markdown. Validation is via embedded checklists in each skill (per `ARCHITECTURE.md:204`).

## Security & Performance

**No new trust boundaries introduced.** The security agent operates within the same trust model as all other agents — human input only, no external data ingestion beyond what the researcher does via WebFetch/WebSearch.

**Sensitive path constraint applies.** The security agent inherits the standard prohibition on writing to `.git/`, `~/.ssh/`, `node_modules/`, `.venv/`, `.env`, `.env-*`.

**Ad-hoc mode risk:** When analyzing arbitrary project files, the agent reads potentially sensitive code (auth modules, crypto implementations, config files). This is inherent to security review and acceptable — the agent already runs within the user's permission scope.

**STRIDE framework adds systematic coverage** but could generate noise on simple specs. The skill should calibrate depth to spec complexity — skip STRIDE categories that clearly don't apply (e.g., Repudiation for a pure UI spec).

## Recommendations

**1. Follow the pm/architect agent pattern for the security agent.** Same frontmatter structure, same tool set (`Read, Write, Edit, Glob, Grep, Agent(researcher)`), same second-person persona voice. The security agent is an artifact producer that can spawn the researcher — identical to how pm and architect work.

**2. Model the skill on debug + research hybrids.** Debug's analytical process (read artifact → analyze → produce findings) combined with research's input routing (detect mode from argument type) gives the best structural template.

**3. Create a security-template.md asset for the security.md output format.** This follows the convention of `spec-template.md`, `plan-template.md`, `research-template.md`. The template should include sections for: summary, findings table, STRIDE coverage, and routing recommendations.

**4. Create a security-dod.md reference for the validation checklist.** Follows the convention of `plan-dod.md` and `research-dod.md`. Covers: all findings have severity + suggestion + route, STRIDE sweep completed, no duplicate of existing ARCHITECTURE.md security considerations, alignment with locked constraints.

**5. Calibrate STRIDE depth to spec complexity.** Not every spec needs all six STRIDE categories analyzed. The skill should identify which categories are relevant based on the spec's data flows and trust boundaries, and skip categories that clearly don't apply. This prevents noise on simple specs while maintaining thoroughness on complex ones.

**6. Consider how security.md interacts with plan validation.** The plan DoD already checks security boundaries (items 17–18). When `security.md` exists, the architect could reference it during planning. This is an opportunity for future integration — not required for the initial build, but worth noting in the skill's documentation.

## Peer Feedback

**For Architect:** The plan should account for two deliverables (agent + skill) plus two assets (`security-template.md`, `security-dod.md`). The meta-agent and meta-skill patterns from spec 001-meta provide the build template. Task ordering: template and DoD first (they're referenced by the skill), then skill, then agent.
