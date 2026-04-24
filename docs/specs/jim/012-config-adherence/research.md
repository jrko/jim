---
spec: "spec.md"
status: Needs PM Review
date: "2026-04-22"
---

<!-- Budget: <1500 words total. Never paste >20 lines of code — use file:line-range + 1-sentence summary. -->

# Research: Config adherence — placeholder rewrites, shared resolve-paths preamble, schema validation

## Anchors

**Files to modify (placeholder rewrites):**

- `/home/jrko/src/jim/skills/build/SKILL.md` (L99–100) — completion gate references `ARCHITECTURE.md` and `BACKLOG.md` with parenthetical hedges; this is the confirmed reproduction site of the observed failure.
- `/home/jrko/src/jim/skills/plan/SKILL.md` (L54–56, L91, L100, L127–128) — four separate prose references to `ARCHITECTURE.md`, `research.md`, and `plan.md` by literal name.
- `/home/jrko/src/jim/skills/vision/SKILL.md` (L30, L36, L79) — three inline references to `VISION.md` and `ARCHITECTURE.md`.
- `/home/jrko/src/jim/skills/arch/SKILL.md` (L23–24, L34, L38, L45, L93) — `ARCHITECTURE.md` and `VISION.md` referenced directly throughout.
- `/home/jrko/src/jim/skills/backlog/SKILL.md` (L16, L40, L64, L69, L81, L152, L160) — seven references to `BACKLOG.md`, `VISION.md`, and spec/plan paths.
- `/home/jrko/src/jim/skills/brainstorm/SKILL.md` (L31) — `VISION.md` and `ROADMAP.md` inline.
- `/home/jrko/src/jim/skills/sec/SKILL.md` (L52, L60) — `ARCHITECTURE.md` and `VISION.md` with parenthetical hedges matching the failure pattern.
- `/home/jrko/src/jim/skills/spec/SKILL.md` (L38–39, L131) — `VISION.md`, `ARCHITECTURE.md`, output path.
- `/home/jrko/src/jim/skills/research/SKILL.md` (L39, L96) — output path and strategic doc references.
- `/home/jrko/src/jim/skills/roadmap/SKILL.md` (L30) — `VISION.md` and `ROADMAP.md`.
- `/home/jrko/src/jim/agents/pm.md` (L50–53) — context paths listed with literal filenames.
- `/home/jrko/src/jim/agents/architect.md` (L49–52, L63) — strategic doc paths listed with literal names.
- `/home/jrko/src/jim/agents/researcher.md` (L50–52, L60) — literal filenames in context and principles.
- `/home/jrko/src/jim/agents/security.md` (L50–53, L62–63) — literal filenames throughout.
- `/home/jrko/src/jim/agents/coder.md` (L49, L56) — spec/plan/research paths inline.
- `/home/jrko/src/jim/agents/meta.md` (L53–54) — spec/plan paths inline.

**Files to create:**

- `skills/_shared/resolve-paths.md` — new; the shared preamble.
- `skills/_shared/config-schema.md` — new; canonical key registry with defaults and value constraints.

**Files to update for schema source-of-truth:**

- `/home/jrko/src/jim/skills/config/SKILL.md` (L33, L44–46, L54) — currently references `assets/config-template.md` as the canonical default source; must defer to the schema post-refactor.
- `/home/jrko/src/jim/skills/config/assets/config-template.md` (L14–44) — the existing defaults table (14 configurable keys across paths, spec ID, workflow gates); the schema must cover all of these.

**Test template:** Jim has no automated test suite (`ARCHITECTURE.md` L215–218). The acceptance criterion for testing is: existing workflow commands function unchanged against projects without `.jim/config.md`. No test file template applies; the coder's validation surface is the static audit (grep) and the dogfood run.

## Local Patterns

**Config resolution pattern (current state):** Every skill that references a configurable path follows one of two prose structures:

1. Inline literal + parenthetical hedge: `ARCHITECTURE.md (default, configurable via .jim/config.md)` — the failure pattern. The agent anchors on the concrete string.
2. Explicit step-1 instruction: "Read `.jim/config.md` if it exists" — present in most skill step 1s but leaves resolution implicit across subsequent steps.

`/jim:backlog` succeeded in the observed failure scenario because it was invoked as a subskill and its step 1 re-read config explicitly. `/jim:build`'s completion gate references `ARCHITECTURE.md` directly at L99 without a re-read trigger, so the parenthetical was skipped.

**Existing defaults:** The canonical default list lives in `/home/jrko/src/jim/skills/config/assets/config-template.md` (L14–44): 14 keys across `path.*` (9 keys), `specs.*` (2 keys), and `workflow.*` (3 keys).

**Skill structure convention:** All SKILL.md files use numbered `### N. Step name` sections. Step 1 is always "Read config" or equivalent. The shared preamble will replace or extend every skill's step 1. Skills stay under 500 lines (`ARCHITECTURE.md` L152).

**`_shared/` directory:** Does not yet exist. `ARCHITECTURE.md` (L26–38) lists 14 skill directories with no `_shared/` entry — the post-build ARCHITECTURE.md refresh must add it.

**Agent context blocks:** Agent files contain a "Context" or "Paths" section listing default filenames as prose (e.g., `agents/architect.md` L49–52). These are read once at agent spawn and are not re-resolved per skill invocation — making them structurally different from skill step-1 resolution. The spec correctly scopes agents to placeholder rewrites only (no preamble for agents).

**`/jim:config` user-facing output:** The config skill necessarily displays literal defaults to the user as part of scaffolding. `/home/jrko/src/jim/skills/config/assets/config-template.md` (L14–44) is the one location where literal default filenames in prose are intentional and must be preserved. The static audit acceptance criterion correctly exempts this file.

## Security & Performance

Security concerns were fully enumerated in `/home/jrko/src/jim/docs/specs/jim/012-config-adherence/security.md`. All four security findings from that review have been incorporated into the spec's acceptance criteria (path value constraints, fenced rendering in the resolved-paths table, error message non-echo policy, ARCHITECTURE.md security labeling). The acceptance criteria at spec lines 37–44 address: traversal via `..` and absolute paths, prompt injection via rendered config values, error-message amplification, and future schema weakening.

No performance concerns: all operations are synchronous markdown reads with no computational overhead.

## Recommendations

**Option A — Preamble as referenced document (preferred).** `skills/_shared/resolve-paths.md` is a standalone prose document. Every skill's step 1 reads: "Follow `skills/_shared/resolve-paths.md`." The preamble instructs the agent to: (1) read `.jim/config.md`, (2) validate all keys against `skills/_shared/config-schema.md`, (3) emit the resolved-paths table before any filesystem call. This is the approach the spec describes and the brainstorm validates as "medium" mechanism strength — a forcing function that produces a checkable artifact.

**Option B — Inline preamble block per skill.** Each skill's step 1 contains the resolution instructions verbatim. Stronger enforcement (no indirection), but 14× maintenance surface. Diverges from the spec's intent and creates the same prose-drift risk the refactor is meant to eliminate.

**Option A is the only architecture-consistent choice.** The spec is explicit; this note is for the Architect to confirm the preamble's invocation syntax (how step 1 references it — `Read and follow` vs. `Invoke` vs. `See`).

**Placeholder syntax:** The spec leaves the exact syntax open (`{path.architecture}` vs. `{{...}}` vs. `${...}`). `{path.architecture}` is the working assumption. The architect should confirm before the build pass begins — a wrong choice means a second search-and-replace across all 14 skills and 6 agents.

**Schema as single source of truth:** `/jim:config`'s template currently holds the canonical defaults. Post-refactor, `config-schema.md` should be that source and the config template should reference it, not duplicate it. The spec's open question (L63) prefers single source of truth — the architect should confirm.

## Peer Feedback

**For PM — Static audit exemption boundary needs precision.** The acceptance criterion states the static audit exempts `skills/_shared/config-schema.md` and `/jim:config`'s "user-facing scaffolding." The config template (`skills/config/assets/config-template.md`) is the artifact that shows defaults to users — it is not SKILL.md itself. The exemption as written could be read to exclude only SKILL.md prose, leaving the template's default table as an audit violation. The spec should clarify that `skills/config/assets/config-template.md` is also exempt from the zero-literal-filenames audit. Without this clarification, the coder either has to remove defaults from the template (breaking the config skill's user-facing output) or fail the static audit check.

**For PM — Agent context sections are structurally different from skill steps.** The spec requires placeholder rewrites in agent files but explicitly excludes agents from invoking the preamble. This is correct — agents are spawned once and do not re-invoke skill steps. However, agent context sections (e.g., `agents/architect.md` L49–52, `agents/researcher.md` L50–52) list paths as documentation for the agent's own reference, not as resolved values used in filesystem calls. A placeholder in an agent body (`{path.architecture}`) has no resolution mechanism — the agent cannot resolve it because it doesn't invoke the preamble. The research finding is that agent placeholder rewrites are documentation-only signals, not enforcement. The spec should acknowledge this gap explicitly, or the PM should confirm that agent rewrites are purely cosmetic alignment (which is a valid, scoped choice — just needs to be stated).

**For Architect — `_shared/` directory convention needs a decision before build starts.** No `_shared/` directory exists anywhere in `skills/`. The preamble will be the first file in a new directory. The architect needs to decide: (1) whether `_shared/` appears in ARCHITECTURE.md's project structure tree before or after the build, and (2) whether the resolve-paths preamble is a SKILL.md (following existing skill conventions) or a plain `.md` (a referenced document, not an invocable skill). This decision affects both the build task for creating the file and the ARCHITECTURE.md refresh acceptance criterion.
