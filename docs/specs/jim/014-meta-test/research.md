---
spec: "spec.md"
status: Active
date: "2026-04-28"
---

<!-- Budget: <1500 words total. Never paste >20 lines of code — use file:line-range + 1-sentence summary. -->

# Research: Meta-test skill — static audit of config-adherence invariants

## Anchors

**Audit surface (the files the skill reads):**

- `skills/*/SKILL.md` — 14 files; see Glob output below
- `agents/*.md` — 6 files: `pm.md`, `architect.md`, `coder.md`, `researcher.md`, `meta.md`, `security.md`

**Shared primitives the skill protects:**

- `skills/_shared/resolve-paths.md` (L1–86) — the canonical preamble; Check 1 verifies every skill references this path inside step 1
- `skills/_shared/config-schema.md` (L1–146) — the schema authority; Check 2 derives its audit surface from the Validation Rules section (L100–108); Check 3 derives the literal-default search corpus from the `keys:` frontmatter (L1–47); Check 4 uses the `keys:` list to identify "configurable path" keys

**Peer skills for structural reference:**

- `skills/meta-skill/SKILL.md` (L1–111) — nearest peer: same `## Process / ### N. Step` structure, same step-1 preamble, same Validate → Present flow
- `skills/meta-agent/SKILL.md` (L1–133) — same structure; shows how a meta-* skill handles gates and validation checklists
- `skills/sec/SKILL.md` (L1–180) — good model for a read-only, judgment-based, no-artifact skill with a structured fail report

**Schema-parsing prior art:**

- `bin/jim_path` (L65–109) — hand-rolled awk parses the `keys:` frontmatter block; shows the exact stable structure the agent will read textually (name → default → type triples, delimited by `---`)

## Local Patterns

### Check 1 — Preamble invocation

All 14 existing `skills/*/SKILL.md` files pass this check today. The invariant pattern is:

```
### 1. Resolve config

Follow `skills/_shared/resolve-paths.md` before proceeding. …
```

Confirmed by Grep across all `skills/*/SKILL.md`. Representative example: `skills/spec/SKILL.md` L20–22, `skills/build/SKILL.md` L30–32. Every occurrence is in `### 1. Resolve config`, not a later step. There are no borderline cases in the current corpus.

**Positive anchor example** (for the skill body): `skills/spec/SKILL.md` L22 — file-path reference `skills/_shared/resolve-paths.md` appears in step 1.

**Negative anchor example shape**: a skill whose step 1 says `### 1. Read the schema` and whose step 2 says `Follow skills/_shared/resolve-paths.md…` — the reference exists but is in step 2, not step 1.

**Edge cases**: Agents (`agents/*.md`) do not have a numbered step structure and do not invoke the preamble directly — they delegate to their preloaded skills. The check applies only to `skills/*/SKILL.md`, not agents. This is confirmed by spec AC and by the agent body structure (agents use a `## Context / ## Process / ## Constraints` flat structure, no `### 1.` step).

### Check 2 — Schema value rules

`skills/_shared/config-schema.md` L100–108 is the exact source of truth for this check. Five rules to audit for presence:

1. **Unknown-key**: "a key present in `.jim/config.md` but not listed in the frontmatter above → hard error" (L102)
2. **file-path / directory-path**: relative; no `..` after normalization; no leading `/`; must resolve within project root after symlinks (L103–104)
3. **positive-integer**: must parse as integer ≥ 1 (L105)
4. **boolean**: literal `true`/`false` only; case variants and YAML 1.1 forms (`yes`, `no`, `on`, `off`) rejected (L106)
5. **string**: any string accepted (L107)

The check verifies presence of each rule in the section, not specific wording. The section heading is `## Validation Rules` (L100).

### Check 3 — Tool-argument literal defaults

**Default values from the `keys:` frontmatter** (config-schema.md L1–47) — these are the literal search terms for the audit:

| Key | Default value |
|-----|--------------|
| `path.vision` | `VISION.md` |
| `path.architecture` | `ARCHITECTURE.md` |
| `path.roadmap` | `ROADMAP.md` |
| `path.workflow` | `WORKFLOW.md` |
| `path.backlog` | `BACKLOG.md` |
| `path.specs` | `docs/specs` |
| `path.brainstorms` | `docs/brainstorms` |
| `path.debug` | `docs/debug` |
| `path.research` | `docs/research` |
| `path.notes` | `docs/notes` |

Non-path keys (`specs.id-padding`, `specs.id-prefix`, `workflow.*`) are not filenames and not subject to this check.

**Current state — no violations found.** Grep across `skills/*/SKILL.md` and `agents/*.md` surfaces only prose mentions:

- `skills/vision/SKILL.md` L4: `VISION.md` in frontmatter `description:` — not a procedural step.
- `skills/arch/SKILL.md` L4: `ARCHITECTURE.md` in frontmatter `description:` — not a procedural step.
- `skills/roadmap/SKILL.md` L4: `ROADMAP.md` in frontmatter `description:` — not a procedural step.
- `skills/backlog/SKILL.md` L5: `BACKLOG.md` in frontmatter `description:` — not a procedural step.
- `skills/brainstorm/SKILL.md` L5: `docs/brainstorms/{YYYYMMDD}-{topic}.md` in frontmatter `description:` — `{YYYYMMDD}-{topic}` suffix makes clear this is illustrative; not a configurable path literal.
- `skills/config/SKILL.md` L37: interview question `"Where do your strategic docs live? (VISION.md, ARCHITECTURE.md, etc.)"` — explicit interview prose inside the scaffolding section; this is the case the `/jim:config` exemption was designed for.
- `agents/security.md` L16: `ARCHITECTURE.md` in a `<commentary>` tag inside an `<example>` block — illustrative example text, not a tool invocation.
- `agents/coder.md` L24: `docs/debug/` in `<commentary>` inside `<example>` — prose.
- `agents/architect.md` L14,24,32: `docs/specs/jim/005-architect/spec.md` and similar — these are concrete spec paths used as `<example>` user inputs, not default filenames.
- `agents/researcher.md` L24: `docs/specs/jim/004-researcher/spec.md` in `<example>` — same.

**Edge cases for the judgment-based check:**

- Frontmatter `description:` fields mentioning `VISION.md`, `ARCHITECTURE.md`, `ROADMAP.md`, `BACKLOG.md` are descriptive metadata, not procedural steps. Clear prose — not violations.
- Agent `<example>` blocks: user-input examples that happen to contain default path names are illustrative, not tool arguments. Clear prose — not violations.
- `skills/config/SKILL.md` L37 interview question: explicitly covered by the `/jim:config` scaffolding exemption. The exemption boundary is the interview/scaffolding section (steps 3–4 of config/SKILL.md), not the whole file. Steps 2 and 5 (which contain Read, Write, Check operations) are subject to normal rules — and they use `.jim/config.md` (not a configurable default filename) as their target.
- The meta-test's own SKILL.md will contain anchor examples (spec AC requires them). The positive anchor example will mention `skills/_shared/resolve-paths.md` in step 1 (fine — not a default filename). Any negative anchor example showing a violation must use a non-default illustrative value or clearly-labeled "bad" snippet to avoid triggering its own check.

### Check 4 — Bash `$({jim_path} <key>)` placeholder substitution

**Current state: zero Bash fenced blocks in any `skills/*/SKILL.md` or `agents/*.md`.** Grep for ` ```bash`, ` ```sh`, ` ```shell` returns no matches. Similarly, `$({jim_path}` and `$(jim_path` return no matches.

This means the initial meta-test run will report `0/0 invocations ✓` for Check 4 — consistent with the spec's pass-case mockup (`N/N invocations ✓`). No existing violations; no existing positive examples in the audit surface to validate against.

The only current `$({jim_path}...)` usage in the codebase is in `skills/_shared/resolve-paths.md` L60 — but that file is excluded from the audit surface.

**What a violation looks like** (for the skill body anchor):

```bash
# Violation — raw helper invocation (loses --root= injection)
cat $(jim_path path.specs)/group/001-name/spec.md

# Violation — hardcoded path
cat docs/specs/group/001-name/spec.md

# Correct form
cat $({jim_path} path.specs)/group/001-name/spec.md
```

**"Configurable path" scoping:** Only paths corresponding to a schema-declared `path.*` key are in scope. An arbitrary relative path like `skills/_shared/config-schema.md` is not a configurable path and does not require `$({jim_path}...)` substitution even in a bash block.

### No test template

Jim has no automated test suite (`ARCHITECTURE.md` L242–244). There is no test file to serve as framework template. The plan's verification approach for this skill will be manual exercise (run the skill, observe the report). The `bin/jim_path` verify battery in `docs/specs/jim/013-jim-path-helper/plan.md` is the closest prior art for exercise-under-fixture methodology, but it is out of scope for this spec.

## Security & Performance

- **Read-only, no writes.** The skill reads skill and agent markdown files and produces an in-conversation report. No filesystem mutations, no Bash invocations in the skill's own runtime path.
- **False-positive bias is a safety property, not a burden.** Check 3's judgment bias toward false-positive means a contributor cannot accidentally pass a lint with an ambiguous literal — they must either rephrase or use a non-default value. The bias is load-bearing.
- **Self-audit is a tamper-resistance property.** The spec requires `skills/meta-test/SKILL.md` to be included in its own audit. If the meta-test's own step 1 omits the preamble reference, Check 1 catches it immediately on first run. This prevents the meta-skill from being a blind spot in the enforcement surface.
- **Schema-derivation is stable.** The `keys:` frontmatter of `config-schema.md` has a documented format constraint (`bin/jim_path` L71–109 shows the parse contract). Adding a key adds a row; removing a key removes one. The agent reads the section textually without needing an awk parser — Claude reads markdown natively. The stable structure means Check 3's literal-default corpus is always derived, never hardcoded in the skill.
- **Agent prose does not need Check 4 today, but the spec says it will.** Agents currently contain no Bash fenced blocks. The check is future-proofing: if a future agent adds a Bash block referencing a configurable path, Check 4 catches it. The audit surface (`agents/*.md`) is correct.

## Recommendations

For the architect, the following options and trade-offs are worth considering:

**Skill structure**: Mirror `skills/sec/SKILL.md` — a read-heavy, judgment-based, no-artifact skill with a structured per-finding report. Sec has the same shape: read context, apply judgment per category, report results, stop. It also shows how to present a pass/fail table alongside per-finding details.

**Deriving the literal-default corpus**: The skill body should instruct the agent to read `skills/_shared/config-schema.md`'s `keys:` frontmatter at runtime to build the list of default values to search for, not hardcode the list. The 10-entry list is stable but will grow as the schema evolves. This mirrors how the resolve-paths preamble reads the schema rather than encoding defaults.

**Exemption boundary for `/jim:config`**: The natural boundary is `skills/config/SKILL.md` steps 3–4 (interview and generate config sections). Step 2 (check for existing config — reads `.jim/config.md`) and step 5 (present and stop) are not scaffolding and are subject to normal Check 3 rules. The plan can pin this to section anchors.

**Check ordering for report efficiency**: Running Check 1 (preamble) first is cheapest — it's a file-path grep. Check 2 (schema rules) is one file read. Check 3 (literal defaults) requires reading all 14 skills + 6 agents. Check 4 (bash substitution) can be combined with Check 3's read pass. Suggest Check 1 → Check 2 → Check 3+4 in a single pass over the audit surface.

**Anchor examples in the skill body**: The spec requires both a positive and a negative anchor example for the preamble check and the literal-default check. For the negative literal-default example, use a fictional skill name and a non-default illustrative path to avoid the meta-test flagging its own body.

## Alignment

This skill aligns with the VISION.md goal of "maintaining architectural consistency" by enforcing the structural discipline that keeps jim's own internals adherent to the config-mediated filesystem safety model. It directly fulfills the ARCHITECTURE.md Security Considerations commitment (L237): "Static-audit pass at refactor close-out verifies current skill bodies invoke the preamble; ongoing enforcement that future skills do the same is deferred to a self-test meta-skill (tracked in `BACKLOG.md`)." The meta-test is the named deferred item; this spec closes that gap. No divergence from locked constraints.

## Peer Feedback

No plan exists yet for this spec. No plan-invalidation signals.

**For PM (spec observation — advisory, not blocking):**

Check 4 will always report `0/0 invocations ✓` on the initial run because no existing skill or agent contains a Bash fenced block referencing a configurable path. This is the correct result — there is nothing to flag. The check is infrastructure for future contributors. The pass-case mockup in the spec already anticipates this with `N/N` rather than a fixed count. No spec change needed, but the plan should note this so the coder writes an appropriate step-1 note and the test exercise confirms the 0/0 case explicitly.
