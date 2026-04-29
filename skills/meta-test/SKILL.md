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
