---
spec: "spec.md"
status: Needs PM Review
date: "2026-04-26"
---

# Research: jim_path helper — shell-mediated config-adherent path resolution for Bash calls

## Anchors

- **`skills/_shared/resolve-paths.md` (L1–75)** — current preamble. Step 2 reads `.jim/config.md` from the project root via `$PWD`; no plugin-root concept exists. Spec 013's placeholder extension lives here.
- **`skills/_shared/config-schema.md` (L1–123)** — schema with 16 keys; `keys:` frontmatter (L2–48) is what the helper validates against. The "Derived placeholders" section the spec mandates does not yet exist.
- **`skills/build/SKILL.md` L73** — only existing skill that runs `Bash` against a project-specific script (`./pre-commit.sh`); not a configurable path. Useful pattern reference but not a sweep target.
- **`skills/meta-agent/SKILL.md`** — references Bash conceptually but does not invoke it (meta-agent builds markdown, not code).
- **`agents/meta.md` L46** — prose line "Jim plugin root: the project root where you are invoked" conflates plugin root with project root; only place in the codebase that mentions a "plugin root" concept, and it's a confused mention.
- **`bin/jim_path` (to be created)** — primary new artifact.
- **`docs/specs/jim/012-config-adherence/`** — direct precedent. Established the placeholder pattern, the resolve-paths preamble, and the schema; this spec extends them.

## Local Patterns

- **Test template:** Jim has no automated test suite (`ARCHITECTURE.md` L228–233). Validation is by manual workflow exercise — see spec 012's dogfood acceptance criterion as the precedent for this spec's own dogfood AC.
- **Path-resolution pattern:** Skills' step 1 invokes `skills/_shared/resolve-paths.md`. The preamble's resolved map is held internally and substituted at point of use (`resolve-paths.md` step 5). New `{jim_path}` placeholder must follow the same lifecycle.
- **Schema authority:** Every config-related decision routes through `config-schema.md`'s frontmatter `keys:` list (`resolve-paths.md` step 1, step 3.1). The helper's key-name validation must read the same source — duplicating the key list in the helper would be the drift risk security finding 4 names.
- **Overlay boundary:** `skills/_shared/` is not overlayable (`config-schema.md` L100–122). The helper's invocation contract is plugin contract, not user-customizable.
- **No executable precedent:** Zero `bin/`, `scripts/`, or shell artifacts exist in the repo today. `bin/jim_path` is the first.

## Static-audit baseline

Sweep target inventory across `skills/**/SKILL.md` for Bash invocations referencing literal default config-key paths: **zero violations**. The grep for `\`\`\`bash` and `Bash(` in skill bodies returns no matches; literal default filenames appear only in (a) `config-schema.md` and `/jim:config` (expected exceptions per the spec), (b) templates and references (`assets/`, `references/`), and (c) skill prose narrative — never inside Bash invocations. The sweep AC is therefore **preventative, not corrective** — the baseline is clean. The static-audit AC's value is forward-looking: it locks in the clean baseline before any future skill introduces a Bash literal-default antipattern.

## Prior Art

No published Claude Code plugin ships executable helpers alongside markdown skills. Jim is establishing the convention. No file-level table applies.

The closest precedent within the codebase is spec 012 itself — it established the `{path.*}` placeholder pattern and the `_shared/` preamble. Spec 013 extends both mechanisms to shell-mediated invocations. The same review patterns (security findings around fencing, trust-contract naming, value/key validation) recur, which is itself signal that the precedent is sound.

## Security & Performance

- **Confirmed: no plugin-root discovery in jim today.** The current preamble has no concept of where the plugin's own files live — only the project root. This is the load-bearing gap the architect must close. *Mitigation surfaces below in Recommendations.*
- **Path-of-discovery is itself security-relevant.** Whichever mechanism the architect picks for plugin path discovery becomes the trust boundary for `bin/jim_path`. A wrong or attacker-influenced helper path means arbitrary execution; the security review's Finding 3 (quoting) and Finding 1 (trust contract) become tighter or looser depending on the discovery mechanism chosen.
- **Performance is non-issue.** Helper invocation is bash subprocess startup (~3–10ms cold) + two file reads (schema + config, both <2KB). Skills make handful of calls per Bash invocation. No caching needed.
- **YAML parsing surface (security finding 4) is plan-phase.** Phase 0 confirms there is no existing YAML-parsing helper or library in the repo; the architect designs from scratch under jim's no-dependency constraint. The recommended grep-line-readable form is the simplest auditable path.

## Recommendations

External research (Claude Code plugin documentation) revealed two platform features the current spec does not exploit. These materially change the design surface and merit PM consideration before the architect commits to a plan.

### Finding A — Plugin `bin/` is auto-added to the Bash tool's PATH

Claude Code's plugin platform documents a `bin/` directory convention: executables placed at `<plugin-root>/bin/` are automatically added to the Bash tool's PATH while the plugin is enabled (https://code.claude.com/docs/en/plugins.md). Skills can therefore invoke `jim_path` by name — no absolute path required, no plugin-root discovery required.

**Implication for the spec:** the current `{jim_path}` placeholder expansion bakes in two values (absolute helper path + absolute project root) but only one is structurally necessary. With PATH-based invocation, the helper resolves itself; the placeholder only needs to convey the project root. Three design re-framings are now on the table:

1. **Drop the placeholder entirely.** Skills write `$(jim_path path.architecture)` directly. Helper reads `$PWD` for the project root. Simplest possible design; preserves the user's monorepo cd-concern as a known weakness (helper finds no config in subdirs → silent fallback to defaults).

2. **Slim the placeholder to project-root only.** Define `{jim_path}` as expanding to `jim_path --root='<absolute-project-root>'` (single-quoted per security finding 3). Skills write `$({jim_path} path.architecture)`. After substitution: `$(jim_path --root='/abs/project/root' path.architecture)`. PATH resolves the helper; `--root` makes it cd-safe. **Recommended** — it preserves the cd-safety property the user explicitly asked for while removing the now-unnecessary helper-path baking.

3. **Keep the spec as drafted.** The multi-token expansion form `bash <abs-path> --root=<abs-root>` works regardless of whether `bin/` is on PATH; it's just over-specified. Cost: one more substitution slot, plus a plugin-root discovery mechanism the architect must design. Benefit: independence from Claude Code's `bin/` PATH convention if it ever changes.

### Finding B — `${CLAUDE_SKILL_DIR}` is documented and stable

Claude Code documents `${CLAUDE_SKILL_DIR}` as the skill's own directory, available in bash injection contexts (https://code.claude.com/docs/en/skills.md). The system-reminder line ("Base directory for this skill: …") is **not** documented as stable contract — it's an implementation detail that could change between Claude Code versions. The architect should rely on `${CLAUDE_SKILL_DIR}` (or the `bin/` PATH convention) rather than parsing the system-reminder.

If recommendation A.3 is chosen and plugin-root discovery is still needed, `${CLAUDE_SKILL_DIR}` derives it: strip the trailing `/skills/<skill-name>` to get the plugin root. This is a documented pattern, not a heuristic.

### Recommendation summary for the architect

- **Strongly preferred:** Recommendation A.2 (slim placeholder, `bin/` PATH, project root via `--root`).
- If A.2 is adopted, security finding 3's quoting requirement applies only to the project-root substitution (the helper is on PATH, not absolute-path-substituted).
- Plan-phase decision (security finding 4): YAML parsing strategy. Recommended: restrict schema and config to grep-line-readable form, parse via small awk script — preserves jim's no-dependency stance and shrinks the parser audit surface to <20 lines.

## Alignment

- **VISION alignment:** Roadmap Phase 3 ("Configuration and Integration") explicitly anticipates configuration-adjacent work; this spec is in scope. No vision conflict.
- **ARCHITECTURE alignment:** Spec acknowledges the introduction of jim's first non-markdown executable artifact and amends ARCHITECTURE.md as part of the AC. The `bin/` convention finding (Recommendation A) actually *increases* alignment — the helper uses Claude Code's documented plugin convention rather than jim-specific path tricks.

## Peer Feedback

**For the PM:**

1. **Recommendation A re-frames acceptance criteria.** If A.2 is adopted (slim placeholder), the spec's "Placeholder and preamble" AC block needs revision — the multi-token `bash <abs-path> --root=<abs-root>` expansion form becomes `jim_path --root='<abs-root>'`, and the "absolute helper path" sub-criterion drops out. Security finding 3's quoting requirement remains but applies only to the project root.

2. **The "first executable artifact" framing in the Overview and AC remains accurate** — the helper is still jim's first executable code — but the architectural-shift narrative softens slightly: jim is adopting a documented Claude Code plugin convention (`bin/` on PATH), not inventing a novel pattern. ARCHITECTURE.md update should reflect this distinction.

3. **The static-audit baseline is clean.** No skills currently violate the to-be-introduced rule. The sweep AC is preventative; the dogfood-cycle AC remains the load-bearing acceptance signal. PM should consider whether the AC text reflects this — currently it reads as if there is existing literal-default usage to clean up.

**For the architect:** No plan exists yet, so no plan invalidation. Architect should treat Recommendation A's options as the first design decision for the plan; adoption of A.2 propagates into Contract definitions (the placeholder's expansion shape).
