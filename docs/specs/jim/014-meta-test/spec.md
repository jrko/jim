---
title: "Meta-test skill — static audit of config-adherence invariants"
type: feature
group: "jim"
id: "014"
status: approved
origin:
  - docs/specs/jim/012-config-adherence/spec.md
  - docs/specs/jim/012-config-adherence/security.md
  - docs/specs/jim/013-jim-path-helper/spec.md
  - BACKLOG.md
---

# 014 Meta-test skill — static audit of config-adherence invariants

## Overview
A `/jim:meta-test` skill that audits jim's own skills and agents for config-adherence invariants — preamble invocation, schema value rules, `$({jim_path} <key>)` placeholder substitution, and literal default filenames in tool-argument positions — using Claude's judgment over skill prose rather than deterministic regex. The skill is the forcing function that prevents future drift on the structural defenses 012 and 013 established.

## Problem Statement
Specs 012-config-adherence and 013-jim-path-helper established placeholder discipline and the `bin/jim_path` helper as the structural defenses against literal default filenames leaking into filesystem operations. These defenses are snapshot-only: a contributor writing a new skill (or modifying an existing one) can silently break them by skipping the resolve-paths preamble, omitting value constraints when adding a new path key, writing a literal default filename in a tool-argument position, or making a Bash call to a configurable path without `$(jim_path …)` substitution. ARCHITECTURE.md explicitly defers ongoing enforcement to "a self-test meta-skill (tracked in `BACKLOG.md`)." Without it, drift surfaces only through manual review or through the next user who hits a broken override.

## User Stories
- As a jim contributor, I can run `/jim:meta-test` after touching skills or agents to verify my changes don't drift from the four config-adherence invariants, so I catch leaks before commit rather than during review.
- As a jim contributor reviewing a PR, I can run `/jim:meta-test` to get a structured report of any invariant violations the PR introduces, so review focuses on the failures instead of reading every line for compliance.
- As a jim maintainer editing `config-schema.md` (adding a new key, changing a type rule, or amending the validation surface), I can run `/jim:meta-test` to verify every documented value-type rule (path constraints, `positive-integer`, `boolean`, `string`, unknown-key) is still present.

## Acceptance Criteria
- [ ] `/jim:meta-test` slash-command exists and dispatches to the `@jim:meta` agent (consistent with `/jim:meta-skill` and `/jim:meta-agent`).
- [ ] The skill's audit surface is exactly `skills/*/SKILL.md` and `agents/*.md`. The skill body explicitly excludes `skills/_shared/`, `skills/*/references/`, `skills/*/assets/`, `bin/`, `.claude-plugin/`, and root strategic docs (`VISION.md`, `ARCHITECTURE.md`, `ROADMAP.md`, `BACKLOG.md`, `WORKFLOW.md`, `CLAUDE.md`).
- [ ] The skill applies four invariant checks across the audit surface:
  - **Preamble invocation:** every `skills/*/SKILL.md`'s step 1 contains a literal reference to the file path `skills/_shared/resolve-paths.md`. The check is structural — file-path reference inside step 1 — not phrase-matched, so rephrasing surrounding prose does not break the check, but a missing or relocated reference does. Skill body includes a positive and a negative anchor example.
  - **Schema value rules:** every value-type rule documented in `skills/_shared/config-schema.md`'s Validation Rules section is preserved across schema edits — `path.*` constraints (relative path, no `..` segments after normalization, no leading `/`, resolves within project root), `positive-integer` (≥ 1), `boolean` (literal `true`/`false` only; case variants and YAML 1.1 `yes`/`no`/`on`/`off` rejected), `string`, and the unknown-key hard-error rule. The check verifies presence of each rule, not specific wording.
  - **Tool-argument literal defaults:** no literal default filename from the schema's `keys:` list appears in a tool-argument position (function-call syntax like `Read(...)` / `Glob(...)`, inline backticks governed by a tool-call verb in a procedural step, or fenced bash/shell code blocks). Casual prose mentions, headings, documentation tables, and explanatory paragraphs are not violations — Claude applies judgment about whether a literal is acting as a tool argument or as descriptive prose. **Bias is toward false-positive:** when ambiguous (the literal could plausibly be either prose or a tool argument), flag it. A contributor can resolve a false flag by rephrasing or by using a non-default illustrative value, but a missed leak goes silent. Skill body anchor examples illustrate both clearly-prose and clearly-tool-argument cases to calibrate the judgment.
  - **Bash `$({jim_path} <key>)` placeholder substitution:** every Bash invocation in the audit surface that references a configurable path uses the placeholder form `$({jim_path} <key>)` — not a raw `$(jim_path <key>)` invocation (which loses spec 013's cd-safe `--root=` flag injection), not a hardcoded path, and not an alternative resolution mechanism such as inline `awk` parsing of `.jim/config.md`. `<key>` must be a config key declared in `config-schema.md`'s `keys:` frontmatter list. "Configurable path" means a path corresponding to a schema-declared key, not arbitrary unrelated paths.
- [ ] **Self-audit by inclusion:** the meta-test audits its own `skills/meta-test/SKILL.md` along with all other audited skills. Self-exclusion is prohibited; the meta-test's own preamble invocation, literal-default-filename position, and any Bash invocations are checked under the same rules as every other skill.
- [ ] **`/jim:config` scaffolding exemption (narrow):** literals appearing inside the scaffolding instructions for new `.jim/config.md` files in `skills/config/SKILL.md` are exempt — defined as the section that emits default key/value pairs into the user's project config. Procedural Read/Write/Glob steps elsewhere in `skills/config/SKILL.md` are subject to the same literal-default-filename check as every other audited skill. The plan can pin the exemption boundary to specific section anchors when the skill is built.
- [ ] Output is an in-conversation pass/fail report. On pass, lists per-invariant counts. On fail, lists each violation with file path, the invariant violated, and a quoted excerpt showing the problem so the contributor can locate and fix it.
- [ ] The skill writes no artifacts to disk — report stays in conversation only.
- [ ] The skill is non-mutating: it never modifies skill or agent files, and never auto-fixes violations.
- [ ] The skill follows the standard skill structure: step 1 invokes `skills/_shared/resolve-paths.md`, subsequent steps describe the audit procedure and report format. Skill stays under the 500-line progressive-disclosure budget; methodology detail goes to `references/` if needed.

## UI Mockup
Pass case:
```
✓ /jim:meta-test — all invariants satisfied

  Audited 14 skills, 6 agents.

  - Preamble invocation: 14/14 ✓
  - Schema value rules: 5/5 type rules preserved ✓
  - Tool-argument literal defaults: 0 violations ✓
  - Bash $({jim_path} <key>) placeholder substitution: N/N invocations ✓
```

Fail case:
```
✗ /jim:meta-test — 3 violations across 2 files

  Audited 14 skills, 6 agents.

  Preamble invocation: 13/14 (1 violation)
    - skills/foo/SKILL.md — step 1 does not reference
      skills/_shared/resolve-paths.md

  Tool-argument literal defaults: 2 violations
    - skills/bar/SKILL.md:42 — literal `ARCHITECTURE.md` in
      `Read \`ARCHITECTURE.md\`` (use {path.architecture})
    - agents/baz.md:18 — literal `BACKLOG.md` in Glob argument
      (use {path.backlog})

  Schema value rules: 5/5 type rules preserved ✓
  Bash $({jim_path} <key>) placeholder substitution: 8/8 ✓
```

## Out of Scope
- **Runtime exercise / fixture-project dogfood.** Invoking actual skills against a fixture project with non-default `.jim/config.md` to verify end-to-end behavior. Deferred to a Tier-2 spec; the placeholder mechanism is itself the runtime forcing function for native tool calls, and `bin/jim_path`'s verify battery (spec 013) covers the helper-side runtime check.
- **LLM-as-judge / behavioral evaluations** using promptfoo, DeepEval, Ragas, or similar. The invariants under audit are structural facts about markdown source; behavioral evals would test "does Claude follow correctly written instructions" — a different question.
- **CI integration.** The skill runs in-conversation only. A future spec may add a parallel `bin/` script that executes the same checks for CI.
- **Auditing files outside the consumer surface.** `skills/_shared/`, `skills/*/references/`, `skills/*/assets/`, `bin/jim_path`, `.claude-plugin/`, and root strategic docs are not audited.
- **Anti-pattern lint beyond config adherence** — personality soup, instruction shadowing, line-count limits, frontmatter consistency, file-reference existence, prose anti-patterns. Separate concern.
- **Auto-fix.** The skill reports findings; it never modifies files.
- **Self-modification.** The meta-test never writes back to its own `SKILL.md` or any other audited file. Distinct from self-audit (which is required, per the AC bullet above).
- **Schema-vs-skill cross-reference.** The skill does not check whether every schema key has at least one consumer or whether every consumer-referenced key exists in the schema. Adding/removing keys is `/jim:config` and `/jim:meta-skill`'s concern.

## Open Questions
- [x] ~Should the Bash `$(jim_path …)` substitution check apply to `agents/*.md` as well as `skills/*/SKILL.md`?~ → Yes. Agents (notably coder and security) contain Bash invocation prose; the invariant applies there too. All four checks run across the full audit surface (skills + agents).
- [x] ~Should the fail-case report use a machine-readable shape (e.g., fenced JSON) for future CI consumption?~ → No. Human-readable markdown only. CI concerns belong to the deferred Tier-2 spec.
