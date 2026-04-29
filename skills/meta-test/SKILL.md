---
name: meta-test
description: >
  Statically audit jim's own skills and agents for config-adherence
  invariants — preamble invocation, schema value rules, literal default
  filenames in tool-argument positions, and `$({jim_path} <key>)`
  placeholder substitution in Bash invocations. Use when the user invokes
  /jim:meta-test, before committing changes to skills or agents, or to
  verify the audit surface is clean. Do not use for runtime exercise of
  skills, behavioral evaluation, or auditing files outside the consumer
  surface (skills/_shared/, references/, assets/, bin/, root strategic
  docs are out of scope).
agent: meta
argument-hint: ""
---

# /jim:meta-test

Statically audit jim's own skills and agents for the four config-adherence invariants established by specs 012-config-adherence and 013-jim-path-helper. The skill produces an in-conversation pass/fail report; it writes nothing to disk and modifies no files. Findings rely on Claude's judgment over markdown source — anchor examples in each check section calibrate borderline cases.

*(The `agent: meta` field in this frontmatter is a jim documentation convention, not a Claude Code routing mechanism.)*

## Process

### 1. Resolve config

Follow `skills/_shared/resolve-paths.md` before proceeding. Resolve every `{path.*}`, `{specs.*}`, or `{workflow.*}` placeholder before passing it to a tool call.

### 2. Read the config schema

Read `skills/_shared/config-schema.md` to derive the audit's two key inputs:

- **Configurable-key list and per-key default values** — parsed from the keys: frontmatter at the top of the file (the YAML sequence under `keys:`). This list is the literal-default search corpus for Check 3 and the set of valid keys for Check 4's `$({jim_path} <key>)` substitution form.
- **Type-rule presence checklist** — parsed from the `## Validation Rules` section. The rule names whose presence Check 2 verifies: unknown-key hard-error, file-path/directory-path constraints, positive-integer, boolean, string.

The schema is treated as authoritative input. The meta-test does not maintain its own copy of these lists; tampering with the schema (removing a key, weakening or deleting a rule, corrupting the `keys:` frontmatter) directly affects what the audit can detect. Schema modifications carry the same review weight as modifications to `bin/jim_path` or `resolve-paths.md` — they are part of the security-relevant trio per `ARCHITECTURE.md`.

**Schema-read failure semantics — fail loud, never silently fall back:**

- **Schema file missing or unreadable:** halt with `✗ /jim:meta-test — cannot read skills/_shared/config-schema.md`. Do not proceed to Checks 1–4.
- **`keys:` frontmatter missing or unparseable as a mapping sequence:** halt with `✗ /jim:meta-test — schema keys: section is missing or malformed`. Do not proceed.
- **`## Validation Rules` section missing:** halt with `✗ /jim:meta-test — schema Validation Rules section is missing`. Do not proceed.
- **Schema readable, `keys:` empty (no path keys present):** Check 3 reports `0/0` legitimately; do not halt — the audit can still run with an empty corpus.

Fallback to a hardcoded list, fallback to silent ✓, or fallback to "skip this check" is **prohibited**. Defense-in-depth for the schema-trust boundary documented above: if the schema is broken, the audit is broken, and the user must see that loudly before any other finding.

### 3. Enumerate the audit surface

The audit surface is exactly two file globs:

- `skills/*/SKILL.md` — every skill body
- `agents/*.md` — every agent definition

Glob both patterns. Hold the resulting file list internally for Checks 1, 3, and 4. Check 2 reads only the schema, which was already loaded in step 2.

**Self-audit by inclusion.** `skills/meta-test/SKILL.md` is part of the `skills/*/SKILL.md` glob and is therefore audited along with every other skill. There is no self-exclusion clause and no `if file == 'skills/meta-test/SKILL.md': skip` shortcut. The meta-test verifies its own preamble invocation, its own literal-default-filename position, and any Bash invocations under exactly the same rules as every other skill.

**Out of scope — never audited:**

- `skills/_shared/` — plugin contract, source of truth (schema and preamble live here; auditing them would be circular)
- `skills/*/references/` — methodology documents; may legitimately mention default filenames
- `skills/*/assets/` — templates and fixtures; may legitimately mention default filenames
- `bin/` — executable artifacts; parsed by `bin/jim_path` itself, not by skills
- `.claude-plugin/` — plugin manifest
- Root strategic docs: `VISION.md`, `ARCHITECTURE.md`, `ROADMAP.md`, `BACKLOG.md`, `WORKFLOW.md`, `CLAUDE.md`

These paths are listed for reader reference; they are never globbed and never read by this skill.

### 4. Check 1 — Preamble invocation

**Rule:** every `skills/*/SKILL.md`'s step 1 contains a literal reference to the file path `skills/_shared/resolve-paths.md`.

The check is **structural**, not phrase-matched. Read the first ~40 lines of each skill body — the region where step 1 lives — and look for the literal string `skills/_shared/resolve-paths.md`. Rephrasing the prose around the reference does not break the check; a missing or relocated reference does. Agents (`agents/*.md`) are excluded from this check — they have no numbered step structure and do not invoke the preamble directly.

**Positive anchor (passes):**

```
### 1. Resolve config

Follow `skills/_shared/resolve-paths.md` before proceeding. …
```

The file path appears inside the first step. Any rephrasing of the surrounding prose still passes as long as the reference is present in the step-1 region.

**Negative anchor (fails):**

```
### 1. Read the schema
…
### 2. Resolve paths
Follow `skills/_shared/resolve-paths.md` before proceeding.
```

The reference exists, but it is in step 2, not step 1. The check fails because step 1 has no preamble invocation.

**Per-finding line shape (fail case):**

```
- skills/<name>/SKILL.md — step 1 does not reference
  skills/_shared/resolve-paths.md
```

Count: pass count is `<files-with-reference-in-step-1> / <total-skills-globbed>`. Skills that pass are not listed individually.

### 5. Check 2 — Schema value rules

**Rule:** every value-type rule documented in `skills/_shared/config-schema.md`'s `## Validation Rules` section is preserved across schema edits. The check verifies presence of each rule, not specific wording.

Five rules whose presence is verified:

1. **unknown-key** — a key in `.jim/config.md` not listed in the schema's frontmatter must hard-error.
2. **file-path / directory-path** — values must be relative; must not contain `..` segments after normalization; must not begin with `/`; must resolve within the project root after symlink resolution.
3. **positive-integer** — must parse as an integer ≥ 1.
4. **boolean** — must be the literal YAML `true` or `false`. Case variants (`True`, `FALSE`) and YAML 1.1 forms (`yes`, `no`, `on`, `off`) are rejected.
5. **string** — any string accepted (no constraint beyond the type requirement).

For each rule: read the `## Validation Rules` section's prose and confirm a sentence (or bullet) covers the rule's intent. Wording can vary; intent must be present. A rule can be considered present if the section describes the same constraint a contributor would need to implement.

**Per-finding line shape (fail case):**

```
- Missing rule: <rule name> (e.g., "boolean")
  Section: skills/_shared/config-schema.md ## Validation Rules
```

Count: pass count is `<rules-present> / 5`. The denominator is fixed at 5 — if a future spec adds a sixth type rule, both this skill and the count denominator update together.

### 6. Check 3 — Tool-argument literal defaults

**Rule:** no literal default filename from the schema's `keys:` list appears in a tool-argument position in any audited file. The literal-default corpus comes from step 2's schema read — for each `path.*` key, the `default` field is the string to search for (e.g., `VISION.md`, `ARCHITECTURE.md`, `docs/specs`, `docs/brainstorms`). Non-path keys (`specs.id-padding`, `specs.id-prefix`, `workflow.*`) are not filenames and not subject to this check.

**What counts as a tool-argument position (flag):**

- Function-call syntax: `Read(...)`, `Write(...)`, `Edit(...)`, `Glob(...)`, `Grep(...)` with the literal as an argument
- Inline backticks governed by a tool-call verb in a procedural step — e.g., a step instructing the agent to `` Read `ARCHITECTURE.md` ``
- Fenced bash/shell code blocks where the literal appears as a path argument

**What does not count (pass — descriptive prose):**

- Frontmatter `description:` fields
- Headings, markdown table cells used for documentation
- Plain prose mentions ("the architecture document is read for context")
- `<example>` blocks in agent files (illustrative user input, not tool invocations)
- Backticked references in explanatory paragraphs not adjacent to a tool-call verb

**False-positive bias:** when the literal could plausibly be either prose or a tool argument, **flag it**. A contributor can resolve a false flag by rephrasing or by using a non-default illustrative value, but a missed leak goes silent. Anchor examples below calibrate clear-prose vs clear-tool-argument cases — borderline cases default to flagging.

**Positive anchor — clear prose (passes):**

```
This skill reads ARCHITECTURE.md for locked architectural constraints.
```

The literal appears in a sentence with verbs and articles. No tool-call verb governs it. Pass.

**Negative anchor — clear tool argument (fails):**

```
### 4. Read the architecture document

Read `ARCHITECTURE.md` to verify the change does not violate locked
constraints.
```

The literal sits inside backticks immediately after a tool-call verb (`Read`) in a procedural step. Flag.

(Anchor examples in this skill body deliberately use a fictional path `docs/example/CUSTOM.md` and a fictional skill name when illustrating violations elsewhere, to avoid the meta-test flagging its own examples on self-audit.)

**`/jim:config` scaffolding exemption (narrow):**

The skill `skills/config/SKILL.md` legitimately mentions default values when scaffolding new `.jim/config.md` files. The exemption is pinned to **step 3 and step 4** of `skills/config/SKILL.md`:

- step 3 (interview / discover paths) — exempt
- step 4 (generate config / scaffolding) — exempt

step 1 (resolve config preamble), step 2 (check for existing config), and step 5 (present and stop) of `skills/config/SKILL.md` are subject to normal Check 3 rules. The exemption applies to no other file.

**Per-finding line shape (fail case):**

```
- <file>:<line> — literal `<filename>` in <description of position>
  (use {<schema key>})
```

Count: pass count is `<files-without-violations> / <total-files-audited>`. Each violation is listed individually with file:line.

### 7. Check 4 — Bash placeholder substitution

**Rule:** every Bash invocation in the audit surface that references a configurable path uses the placeholder form `$({jim_path} <key>)` — where `<key>` is one of the configurable keys derived from step 2's schema read.

**What is flagged:**

- **Raw `$(jim_path <key>)`** — drops spec 013's cd-safe `--root=` flag injection. Even if it works under default project root, it silently breaks for any cwd != project root.
- **Hardcoded path values** — `cat docs/example/CUSTOM.md`, `ls docs/specs/group/`, etc., where the literal corresponds to a schema-declared `path.*` default.
- **Alternative resolution mechanisms** — inline `awk` parsing of `.jim/config.md`, `cat .jim/config.md | grep`, or any other shell-mediated config read that bypasses `jim_path`. The helper is the single chokepoint by design.

**What is not flagged:**

- Bash invocations referencing paths that do **not** correspond to a schema-declared `path.*` key (arbitrary paths used in unrelated examples are fine — "configurable path" scope is exactly the schema's path keys).
- Comments or prose inside fenced bash blocks that mention paths textually but are not executed.

**Empty-corpus case (no Bash blocks anywhere in the audit surface):** the check reports `0/0 invocations ✓` rather than omitting the line. The audit must always show all four checks. The empty corpus is the correct result when no skill or agent has yet introduced a Bash block; future contributors adding one will see the count grow.

**Positive anchor (passes):**

```bash
cat $({jim_path} path.specs)/group/001-name/spec.md
```

The placeholder `{jim_path}` will be expanded by the resolve-paths preamble at skill-invocation time to `jim_path --root='<absolute-project-root>'`, producing a cd-safe invocation.

**Negative anchors (fail):**

```bash
# Violation — raw helper invocation (loses --root= injection)
cat $(jim_path path.specs)/group/001-name/spec.md

# Violation — hardcoded path (corresponds to path.specs default)
cat docs/specs/group/001-name/spec.md
```

**Per-finding line shape (fail case):**

```
- <file>:<line> — <description>; use $({jim_path} <key>) form
```

Count: pass count is `<correct-invocations> / <total-bash-invocations>`. Today: `0/0`.
