---
title: "jim_path helper — shell-mediated config-adherent path resolution for Bash calls"
type: feature
group: "jim"
id: "013"
status: approved
origin:
  - BACKLOG.md
  - docs/brainstorms/20260422-config-adherence.md
  - docs/specs/jim/012-config-adherence/spec.md
---

# 013 jim_path helper — shell-mediated config-adherent path resolution for Bash calls

## Overview
Ship a small shell helper (`bin/jim_path`) that prints the resolved value of any `.jim/config.md` key for use in skill Bash calls, eliminating the prose-discipline gap that lets literal default filenames slip into Bash invocations even after spec 012's placeholder rewrites. Introduce a derived `{jim_path}` placeholder so skills write `$({jim_path} path.architecture)` and config adherence becomes a runtime property of the Bash subprocess rather than a write-time judgment by the agent.

## Problem Statement

Spec 012 hardened config adherence for native tool calls (Write, Edit, Read, Glob) by replacing inline default filenames with `{path.*}` placeholders that the agent must resolve before any tool argument. The forcing function works because `{path.architecture}` is not a valid filesystem path — the agent cannot accidentally pass it as-is.

Bash calls have no equivalent forcing function. When a skill writes `test -f ARCHITECTURE.md` in a Bash invocation, the literal default reads naturally and runs without error against any project. Under a non-default config (`path.architecture: docs/jim/ARCHITECTURE.md`), the Bash call silently checks the wrong file. The agent must remember to resolve the placeholder *into* the Bash command at write time — exactly the per-call judgment 012 was built to remove. The pattern that worked for native tool calls does not extend to the Bash surface.

A second related gap: monorepo users run skills from project subdirectories. The `resolve-paths` preamble itself reads `.jim/config.md` from the project root via Claude Code's session-level primary working directory, but a Bash subprocess started in a subdirectory has no analogous anchor — `$PWD` is wrong. Any helper that does its own discovery via cwd or filesystem walk-up either fails (cwd) or risks crossing the project boundary (walk-up).

The helper closes both gaps. `$(jim_path X)` is evaluated by Bash at runtime, so the agent can write the placeholder once and never compute the resolved value itself. The project root flows through the placeholder expansion from the preamble's resolved map, so the helper works regardless of the Bash invocation's cwd.

## User Stories

- As a jim skill author, I can reference configured paths in skill Bash calls without computing the resolved value, so that I cannot accidentally invoke shell tools with the wrong path under custom config.
- As a jim user with a custom `.jim/config.md`, I can trust that every Bash invocation across the skill surface honors my overrides, so that config adherence is uniform and not a per-skill quality concern.
- As a jim user working in a monorepo, I can invoke skills from project subdirectories and have the helper still resolve against my project's `.jim/config.md`, so that workflow does not depend on cwd discipline.
- As a jim contributor adding a new skill, I can use one canonical pattern (`$({jim_path} path.X)`) for any configurable value in any Bash call, so that there is one habit to learn and one pattern to audit.

## Acceptance Criteria

### Helper script

- [ ] `bin/jim_path` exists in the plugin repo with `#!/usr/bin/env bash` shebang.
- [ ] Helper accepts a config key as its first positional argument, full-key form (`jim_path path.architecture`, `jim_path workflow.require-research`). Bare suffixes (`jim_path architecture`) are rejected as unknown keys.
- [ ] Helper accepts `--root <absolute-path>` flag specifying the project root. When omitted, the helper falls back to `$PWD`.
- [ ] Helper resolves keys for every entry in `skills/_shared/config-schema.md`'s `keys:` frontmatter — `path.*`, `specs.*`, and `workflow.*` keys are all supported.
- [ ] Helper validates the requested key against the schema's `keys:` frontmatter. An unknown key (typos, omitted keys, removed keys) exits non-zero with a one-line stderr error in the form `jim_path: unknown key (see .jim/config.md)`. The offending key is **not** echoed in the error: Claude Code's Bash tool captures subprocess stderr into agent context, and an attacker-controlled key name (via a malicious committed `.jim/config.md` in a cloned repo) could otherwise flow markdown, prompt-injection content, or terminal escape sequences into that context. The helper does not echo raw values at any point either. Operators diagnose unknown-key errors by reading `.jim/config.md` directly.
- [ ] Helper does **not** validate the *value* in `.jim/config.md` against schema constraints. The resolve-paths preamble validates values at skill-invocation time; the helper trusts what's there. Re-validating in the helper is duplication and a drift risk.
- [ ] If the resolved key is absent from `.jim/config.md`, or if `.jim/config.md` does not exist at the resolved root, the helper prints the schema default to stdout and exits 0.
- [ ] Helper prints exactly the resolved value to stdout, followed by a single trailing newline. Nothing else is written to stdout.
- [ ] Exit codes: `0` on success, `2` on unknown key, `1` on any other error (config unreadable, schema unreadable, malformed YAML).
- [ ] **Trust contract:** the helper assumes the resolve-paths preamble has validated `.jim/config.md` against schema constraints prior to any helper invocation. The helper itself does not re-validate values; it trusts what the preamble has already accepted. Skills that invoke the helper without first running the preamble violate this contract. Sustained enforcement that future skills run the preamble is tracked in `BACKLOG.md` (Self-test meta-skill enforces preamble invocation); this spec relies on that backlog item for defense in depth and does not duplicate the enforcement here.

### Placeholder and preamble

- [ ] `skills/_shared/resolve-paths.md` is extended to compute and substitute a derived `{jim_path}` placeholder. The placeholder's resolved expansion is a multi-token command of the form `jim_path --root=<absolute-project-root>`. **Helper discovery uses Claude Code's documented plugin `bin/` convention**: the plugin's `bin/` directory is automatically added to the Bash tool's `PATH` while the plugin is enabled, so the helper is invoked by name with no absolute-path computation required. The spec deliberately leverages this stable platform feature rather than introducing a custom plugin-root discovery mechanism.
- [ ] **Quoting:** the placeholder expansion shell-quotes the absolute project root using single-quotes. Embedded single-quotes are escaped via the `'\''` idiom. The expansion form after quoting is `jim_path --root='<project-root>'`. The preamble halts with a validation error if the project root cannot be safely single-quoted (e.g., contains a null byte). This closes the metacharacter-injection vector at the placeholder layer rather than inside the helper — the placeholder is the single chokepoint between Claude's environment and the Bash subprocess, so quoting at the chokepoint protects every downstream caller.
- [ ] The preamble determines the absolute project root from Claude Code's session-level primary working directory. The preamble does **not** parse the skill-invocation system-reminder line (`Base directory for this skill: …`) for any path-discovery purpose — that line is not documented as stable contract. If a future change forces plugin-root computation, the preamble derives it from `${CLAUDE_SKILL_DIR}` (the documented stable env var per Claude Code's skills documentation) by stripping the trailing `/skills/<skill-name>` segment.
- [ ] `{jim_path}` is the first placeholder defined by the preamble whose expansion is a multi-token shell command rather than a single value. The preamble documents this explicitly so future placeholders that expand to commands follow the same shape.
- [ ] `skills/_shared/config-schema.md` documents `{jim_path}` in a "Derived placeholders" section adjacent to the `keys:` frontmatter, naming it as a non-configurable, preamble-computed value. Users cannot override `{jim_path}` via `.jim/config.md`.

### Skill sweep

- [ ] **Baseline preserved.** Phase 0 archaeology (see `research.md`) confirmed that current skill bodies contain zero Bash invocations referencing literal default config-key filenames. The sweep AC is therefore preventative: it locks in the clean baseline before any future skill introduces a Bash literal-default antipattern. No retroactive code changes to existing skills are required by this AC; only the static-audit gate.
- [ ] Any Bash invocation in skill bodies (`skills/**/SKILL.md`, current or future) that references a configurable path uses `$({jim_path} <key>)` substitution rather than a literal default filename or a `{path.*}` placeholder embedded inside Bash.
- [ ] Static audit: a grep across `skills/**/SKILL.md` for the literal default filenames defined in `skills/_shared/config-schema.md` (e.g. `ARCHITECTURE.md`, `BACKLOG.md`, `VISION.md`, `ROADMAP.md`, `WORKFLOW.md`, `docs/specs`, `docs/brainstorms`, `docs/debug`, `docs/research`, `docs/notes`) finds zero hits inside Bash code blocks or `Bash(...)` tool-call examples. Expected exceptions: `skills/_shared/config-schema.md` itself and `/jim:config`'s user-facing scaffolding.
- [ ] No skill prose contains the absolute path to `bin/jim_path`. The only handle skills have on the helper is the `{jim_path}` placeholder; helper discovery happens at Bash-execution time via the plugin's `bin/` directory being on `PATH`.

### Dogfood

- [ ] One full `/jim:spec` → `/jim:plan` → `/jim:build` cycle executed against a test project with **all three** of the following present simultaneously:
    1. A `.jim/config.md` overriding at least one `path.*` key (e.g., `path.architecture: docs/jim/ARCHITECTURE.md`).
    2. A `.jim/config.md` overriding at least one non-`path.*` key (e.g., `workflow.require-research: true`) to verify helper parity beyond path keys.
    3. The skill invoked from a non-default cwd (a subdirectory of the project root) at least once during the cycle, to verify the `--root` mechanism works when `$PWD` differs from the project root.
- [ ] Dogfood verification inspects commits/diffs (not just the conversation transcript) to confirm every Bash invocation across the cycle honored the overrides.

### Architecture and documentation

- [ ] `ARCHITECTURE.md` is updated post-build to document:
    - `bin/jim_path` as the project's first non-markdown executable artifact, shipped under Claude Code's documented plugin `bin/` convention (auto-added to the Bash tool's `PATH`). Jim is adopting an existing platform pattern, not inventing a novel one.
    - The `{jim_path}` placeholder and its expansion contract: `jim_path --root='<project-root>'`.
    - The amendment to the previous "no build step, no executable code, pure markdown" claim — the helper is a single bash script with no dependencies and no build step, but it *is* executable code.
- [ ] `skills/_shared/config-schema.md` and `skills/_shared/resolve-paths.md` are flagged in the architecture refresh as remaining the security-relevant files for config validation; `bin/jim_path` is flagged as security-relevant for filesystem access via Bash, with the invariant that it never validates values (the preamble does) and never echoes raw values in stderr.

### Backwards compatibility

- [ ] Existing workflow commands continue to function against projects without `.jim/config.md` and produce artifacts at default paths. (Jim has no automated test suite — interpret as: existing skill behavior unchanged for default-config projects.)
- [ ] The skill sweep does not change any skill's user-visible behavior under default config — only the internal mechanism by which Bash invocations resolve paths.

## Out of Scope

- **Self-test meta-skill enforcing helper adoption.** The static-audit acceptance criterion is a one-shot check at refactor close-out. Continuous enforcement that future skills route Bash calls through `{jim_path}` is deferred to the Self-test meta-skill backlog item.
- **Other Config adherence v2 backlog items.** Resolved-paths audit log and the two ad-hoc enforcement items (preamble invocation, schema value constraints) are tracked separately and do not gate this spec.
- **Native tool calls (Write, Edit, Read, Glob).** These cannot shell-substitute. Spec 012's `{path.*}` placeholder discipline remains load-bearing for those surfaces. The helper is strictly a Bash-surface mechanism.
- **Cross-shell compatibility for the helper script itself.** The helper uses `#!/usr/bin/env bash` and is invoked via Claude Code's Bash tool, which always runs bash. Users invoking `jim_path` from fish or zsh interactive prompts is not a supported case.
- **Windows non-WSL support.** The helper inherits the same bash dependency as Claude Code itself. No additional platform support work in this spec.
- **Manual PATH installation.** Users do not modify their shell PATH. Helper discovery relies entirely on Claude Code's documented plugin `bin/` convention, which adds `<plugin>/bin/` to the Bash tool's PATH automatically while the plugin is enabled. No installer, no symlink, no user setup.
- **Caching of resolved values.** The helper reads `.jim/config.md` and the schema on every invocation. No daemon, no cache file, no memoization.
- **Public-API stability for user shell scripts.** The helper is plugin-internal tooling. Users who want to query their config from their own shell read `.jim/config.md` directly.
- **Migration tooling.** Skills are updated by direct edit. No migration script for projects with custom configs (the existing config format is unchanged).
- **`{plugin.root}` as a general-purpose placeholder.** This spec introduces only the focused `{jim_path}` placeholder. A broader `{plugin.root}` placeholder that exposes other plugin internals to skills is explicitly not introduced here.

## Open Questions

- [ ] Whether the helper's missing-config case (no `.jim/config.md` at the resolved root) should be silent (defaults to stdout, exit 0) or emit a one-line stderr note for diagnostics. Working assumption: silent. Decision deferred to the architect.
- [ ] Exact stderr format for unknown-key errors — short one-line (`jim_path: unknown key: <key>`) vs. matching the preamble's three-line halt format. Working assumption: short one-line, since the helper is a Bash subprocess, not the agent's main control flow. Decision deferred to the architect.
- [x] ~Whether `bin/jim_path` should be committed with the executable bit set~ → Resolved by the move to Claude Code's `bin/` PATH convention: yes, the executable bit must be set. PATH-loaded scripts require it, and the plugin convention is the spec's chosen helper-discovery mechanism.
- [ ] Whether the helper should refuse to run when `.jim/config.md` exists but is unreadable (permissions error), vs. fall back to defaults. Working assumption: refuse and exit 1. Decision deferred to the architect.
