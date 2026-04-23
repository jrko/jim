---
spec: "docs/specs/jim/011-config/spec.md"
status: Active
date: "2026-04-17"
---

# Research: Project Configuration and Overlay

## Anchors

**Config reading hook points** — skills with "read strategic context" steps where `.jim/config.md` reading would be added:

| Skill | Step | Currently reads | Lines |
|-------|------|----------------|-------|
| spec | Step 2 "Read strategic context" | VISION.md, ARCHITECTURE.md | L30–37 |
| plan | Step 2–3 "Handle research" / "Check ARCHITECTURE.md" | research.md, ARCHITECTURE.md | L38–55 |
| research | Step 6 "Alignment Validation" | VISION.md, ARCHITECTURE.md | L88–95 |
| build | Step 2 "Load context" | spec.md, research.md, ARCHITECTURE.md | L40–42, L93 |
| debug | (no strategic context step) | — | — |
| arch | Step 2 "Read VISION.md" | VISION.md | L32–37 |
| vision | Step 2 "Read context" | ARCHITECTURE.md | L24–28 |
| roadmap | Step 2 "Read context" | VISION.md | L24–28 |
| brainstorm | Step 2 "Read context (light)" | VISION.md, ROADMAP.md | L25–27 |
| backlog | Step 1 "Read strategic context" | VISION.md | L34–38 |
| sec | Step 2 "Read architectural context" | ARCHITECTURE.md, VISION.md | L46–56 |
| meta-skill | Gates 1–3 | specs/, research.md, plan.md | L20–40 |
| meta-agent | Gates 1–3 | specs/, research.md, plan.md | L20–40 |

**Asset/reference read points** — where overlay resolution hooks in:

| Skill | Asset | Reference |
|-------|-------|-----------|
| spec | `assets/spec-template.md` (L117) | `references/spec-types.md` (L39, L134) |
| plan | `assets/plan-template.md` (L62, L66) | `references/plan-dod.md` (L100) |
| research | `assets/research-template.md` (L108) | `references/research-dod.md` (L116) |
| build | — | `references/tdd-guide.md` (L16) |
| debug | `assets/debug-template.md` (L62) | — |
| arch | `assets/architecture-template.md` (L44, L64) | — |
| vision | `assets/vision-template.md` (L75) | — |
| roadmap | `assets/roadmap-template.md` (L69) | — |
| sec | `assets/security-template.md` (L136) | `references/security-dod.md` (L131) |
| backlog | `assets/backlog-template.md` (L154) | — |
| brainstorm | — | — |
| meta-skill | — | — |
| meta-agent | — | — |

**Workflow gate locations** — where configurable gates already exist or would be added:

| Gate | Skill | Current behavior | Config key |
|------|-------|-----------------|------------|
| Plan approval | build (L32–35) | Hard gate: stops on `status: draft` | `require-plan-approval` |
| Research existence | plan (L38) | Soft: auto-spawns researcher if missing | `require-research` |
| Research quality | meta-skill (L26), meta-agent (L26) | Hard gate: 7-point spot-check | (already enforced) |
| Security review | spec (step 10), plan (L108–112), sec skill | Soft: offered before spec and plan approval | `require-security` |

**New files to create:**

- `skills/config/SKILL.md` — the `/jim:config` skill
- `skills/config/assets/config-template.md` — scaffolding template for `.jim/config.md`

## Local Patterns

**Path reference style:** Skills use inline string paths, not variables — e.g., `Read VISION.md` or `Write to docs/specs/{group}/{00X}-{name}/spec.md`. Config changes mean editing these strings in each SKILL.md.

**Asset/reference read style:** Consistent pattern across all skills: `"Read assets/{name}.md"` with a relative path from the skill directory. Overlay adds a preceding check: `"Check .jim/skills/{skill}/assets/{name}.md first"`.

**Strategic context step pattern:** Most skills read strategic docs in Step 2, early in the process. Config reading slots in naturally here — one additional file read before any artifact creation.

**Test template:** Jim is a pure-markdown plugin with no executable code or test suite. Validation is via checklists embedded in skills. No test file to reference.

**Skill creation pattern:** See `skills/brainstorm/SKILL.md` as the simplest existing skill (no assets, no references, minimal process). The `/jim:config` skill follows the same create-or-update pattern as `/jim:vision` and `/jim:roadmap`.

## Security & Performance

**No secrets exposure risk.** Config contains paths and boolean flags — no credentials, tokens, or sensitive data.

**Path traversal concern:** Config paths are relative to project root. A malicious config could point to paths outside the project (e.g., `vision: ../../../etc/passwd`). Low risk — the user writes their own config, and Claude Code's sandbox limits file access. Note in documentation that paths should stay within the project root.

**Token cost:** Reading `.jim/config.md` adds one file read per skill invocation. The file is small (under 50 lines of frontmatter). Negligible impact.

**Overlay staleness:** Custom assets/references in `.jim/` won't pick up upstream changes. Document this as a user responsibility — not a runtime concern.

## Recommendations

**1. Config reading instruction pattern.** Add a brief instruction to each skill's strategic context step:

> "Read `.jim/config.md` from the project root if it exists. Use configured `path.*` values instead of the defaults shown in this skill. If the file doesn't exist or a key is omitted, use the paths written in this skill."

This is ~2 sentences added to 13 skills. Skills without a strategic context step (debug) get a new lightweight step.

**2. Overlay instruction pattern.** For the 10 skills that read assets or references, modify the read instruction:

> "When reading `assets/{name}.md`, first check `.jim/skills/{skill-name}/assets/{name}.md`. If it exists, use it instead. Otherwise use the built-in."

Same pattern for `references/`.

**3. Workflow gates.** Two distinct modification patterns:
- **`require-plan-approval`**: Modify existing gate in build (L32–35) to check config value. Current behavior is the default (`true`).
- **`require-research`**: Modify plan skill's research handling (L38). Currently soft (auto-spawns researcher). Config `true` makes it a hard stop instead.
- **`require-security`**: New hard gate in build skill. Security review is currently offered as a soft prompt in both the spec skill (step 10) and plan skill (L108–112) before approval. Config `true` adds enforcement at build time — build refuses to proceed without `security.md` in the spec directory.

**4. Spec ID format.** Modify spec skill's ID assignment logic (L115) to read `specs.id-padding` and `specs.id-prefix` from config. Current default: 3-digit zero-padded, no prefix.

**5. Implementation order suggestion for the architect:**
- Task 1: Create `/jim:config` skill + template (new files, no existing modifications)
- Task 2: Add config reading to all 13 skills (bulk modification, single pattern)
- Task 3: Add overlay resolution to 10 skills with assets/references (bulk modification, single pattern)
- Task 4: Add configurable workflow gates to build and plan skills (targeted modifications)
- Task 5: Add spec ID format config to spec skill (targeted modification)
- Task 6: Update meta agent's `skills:` frontmatter to include `config`
