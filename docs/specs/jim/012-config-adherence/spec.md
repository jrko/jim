---
title: "Config adherence — placeholder rewrites, shared resolve-paths preamble, schema validation"
type: refactor
group: "jim"
id: "012"
status: approved
origin:
  - docs/brainstorms/20260422-config-adherence.md
---

# 012 Config adherence — placeholder rewrites, shared resolve-paths preamble, schema validation

## Overview
Rewrite every skill and agent prose reference to configurable paths as `{path.*}` placeholders, introduce a shared resolve-paths preamble that skills invoke on entry, and add a centralized config schema so malformed or mistyped `.jim/config.md` entries fail fast instead of falling back silently.

## Refactor Rationale

- **Motivation:** The `.jim/config.md` mechanism shipped in spec 011-config is not being reliably honored. Observed failure: during a `/jim:build` completion gate, the agent read the config correctly for `path.backlog` (because `/jim:backlog` was invoked as a subskill and re-read config) but silently skipped config resolution for `path.architecture` and checked the project-root default instead. The failure mode is reproducible because it stems from prose structure: skill bodies reference default filenames inline (e.g. `ARCHITECTURE.md`) with configurability tucked into parenthetical hedges ("default, configurable via `.jim/config.md`"). Agents anchor on concrete strings; parentheticals get skimmed, especially at the tail of long runs where attention is thinnest.
- **Current State:** Default filenames are embedded throughout skill bodies and agent files. Config resolution is a per-reference judgment call with no forcing function. `.jim/config.md` parsing has no schema check — typos (`path.architechture`) silently fall back to defaults, making config errors invisible. Every skill's step 1 repeats "Read `.jim/config.md` if it exists" but then leaves resolution implicit across subsequent steps.
- **Desired State:** Skill and agent prose contains no literal default filenames for configurable paths — only `{path.*}` placeholders. A shared `skills/_shared/resolve-paths.md` preamble, referenced from every skill's step 1, instructs the agent to resolve all placeholders from `.jim/config.md` and emit a resolved-paths table before proceeding. A shared `skills/_shared/config-schema.md` documents every valid config key and its default; unknown keys or malformed values raise a hard error rather than falling through. The resolved-paths table appears inline at the start of every skill invocation — uniformly, not conditionally.
- **Affected Systems:**
  - All skills in `skills/` (14 skills) — placeholder rewrites in procedural prose; step 1 references the shared preamble.
  - All agent files in `agents/` — placeholder rewrites in prose references to configurable paths.
  - New: `skills/_shared/resolve-paths.md` — the shared preamble.
  - New: `skills/_shared/config-schema.md` — the config schema (valid keys, defaults, value constraints).
  - `skills/config/` (`/jim:config`) — updated to reference the shared schema as the source of truth for valid keys and defaults.
  - `ARCHITECTURE.md` — updated post-build to document the `_shared/` directory and the resolve-paths pattern.

## Acceptance Criteria

- [ ] Every configurable path reference in skill bodies (`skills/**/SKILL.md`) uses `{path.*}` placeholder syntax. No literal default filenames appear in skill procedural prose.
- [ ] Every configurable path reference in agent files (`agents/*.md`) uses `{path.*}` placeholder syntax. No literal default filenames appear in agent procedural prose.
- [ ] `skills/_shared/resolve-paths.md` exists and defines: (a) how the agent resolves `{path.*}` placeholders from `.jim/config.md`, (b) the format of the resolved-paths table, (c) instruction to emit the table before any filesystem call.
- [ ] `skills/_shared/config-schema.md` exists and documents every valid `.jim/config.md` key with its default value and expected value shape.
- [ ] Every skill's step 1 references `skills/_shared/resolve-paths.md` as the entry point for config resolution.
- [ ] When `.jim/config.md` contains a key not present in `skills/_shared/config-schema.md`, the preamble instructs the agent to halt with a clear error message naming the offending key. Fallback to defaults is prohibited.
- [ ] `skills/_shared/config-schema.md` defines value constraints per key. For every `path.*` key the schema mandates: (a) the value is a relative path (no leading `/`), (b) after normalization the value contains no `..` segments, (c) the resolved absolute path lies within the project root. Violations raise a hard error via the same path as unknown keys. Non-path keys have documented shape constraints appropriate to their purpose.
- [ ] Validation error messages name the offending key and a brief human-readable description of the violation. Error messages do not echo the raw offending value inline. If a value must be shown for diagnostics, it is rendered inside a fenced code block.
- [ ] Every skill invocation emits a resolved-paths table inline before performing any filesystem-touching work, regardless of whether the skill itself references configurable paths.
- [ ] The resolved-paths table renders each resolved value inside a fenced code block (or equivalent per-value escaping) so that markdown directives or injection payloads within config values cannot be interpreted as agent-addressable content.
- [ ] Agent files do not invoke the resolve-paths preamble directly — the preamble is a skill-invocation-time artifact, and agents that need resolved paths receive them via the skills they invoke.
- [ ] **Static audit:** grep across `skills/` and `agents/` finds zero literal default filenames for configurable keys outside of `skills/_shared/config-schema.md` and `/jim:config`'s user-facing scaffolding (the skill that writes the config file necessarily shows defaults to the user).
- [ ] **Dogfood run:** one full `/jim:spec` → `/jim:plan` → `/jim:build` cycle executed against a test project with at least one non-default override (e.g. `path.architecture: docs/jim/ARCHITECTURE.md`), and every filesystem touch across the cycle honors the override. Verified by inspecting commits/diffs, not just the conversation log.
- [ ] Post-build `ARCHITECTURE.md` refresh explicitly identifies `skills/_shared/resolve-paths.md` and `skills/_shared/config-schema.md` as security-relevant files and names their invariants (e.g. "schema is the sole validation authority for `.jim/config.md`; weakening a constraint weakens validation project-wide").
- [ ] Existing tests pass without modification. (Jim has no automated test suite; interpret as: existing workflow commands — `/jim:spec`, `/jim:plan`, `/jim:build`, `/jim:arch`, `/jim:backlog`, etc. — continue to function against projects without `.jim/config.md` and produce the same artifacts as before at default paths.)

## Out of Scope

- **Completion-gate extraction (`/jim:complete`)** — deferred to a separate spec. The current `/jim:build` completion work stays inline; this spec only ensures its config resolution is reliable via the shared preamble.
- **`jim_path` shell helper** — deferred. Native tool calls (Write, Edit, Read, Glob) can't shell-substitute, so prose-level placeholders are load-bearing regardless; the helper is a future addition for the Bash surface.
- **Self-test meta-skill** — deferred. Acceptance here is manual (static audit + one dogfood run).
- **Idempotency guarantees for any extracted completion skill** — not relevant to this spec.
- **Resolved-paths audit log file** (`.jim/logs/resolved-paths.jsonl` or similar) — deferred. This spec emits the table inline only; persistent logging is a later addition.
- **Changes to `.jim/config.md` file format or key set** — the format and the existing keys from spec 011-config are frozen for this refactor. New keys introduced here (if any) are limited to documenting what already exists.
- **Migration tooling for existing projects with custom configs** — the schema validation runs at read time on every invocation; no one-shot migration is required.
- **Agent resolve-paths preamble** — agents get placeholder rewrites but not the preamble itself. If drift is observed in agent-driven path handling post-refactor, a follow-up spec can address it.
- **Continuous enforcement that future skills invoke the preamble** — the static audit verifies current skills at refactor time. It does not prevent future skills from being added that silently skip the preamble and bypass validation. Ongoing enforcement is deferred to the self-test meta-skill (brainstorm item 7).

## Open Questions

- [ ] Exact placeholder syntax — working assumption is `{path.architecture}`, but alternatives (`{{path.architecture}}`, `${path.architecture}`, `<<path.architecture>>`) may read better depending on how they interact with markdown rendering and agent parsing. Decision deferred to the architect.
- [ ] Resolved-paths table format — single-line-per-key vs. grouped, prefix/delimiter style, whether to include the default alongside the resolved value for contrast. Decision deferred to the architect.
- [ ] Whether `skills/_shared/config-schema.md` also serves as the user-facing config documentation (single source of truth), or whether `/jim:config` maintains its own separate user-facing doc that references the schema. Preference is single source of truth; confirm during planning.
