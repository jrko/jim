---
spec: "spec.md"
status: Needs PM Review
date: "2026-04-29"
---

# Research: Configurable build hooks (pre-commit and pre-completion)

## Anchors

**Schema (single source of truth for new keys)**
- `skills/_shared/config-schema.md:2-47` — `keys:` frontmatter list; new entries slot in here.
- `skills/_shared/config-schema.md:36-38` — `specs.id-prefix` entry; closest precedent (`type: string`, `default: ""`).
- `skills/_shared/config-schema.md:58-88` — three existing per-section headings (Path / Spec ID format / Workflow gate); a new "Hook keys" section follows the pattern.
- `skills/_shared/config-schema.md:100-108` — Validation Rules; `string` rule at L106 ("any string accepted").
- `skills/_shared/config-schema.md:110-121` — Schema Format Constraint (single-line scalars; quoting rules).

**Helper (resolves at runtime)**
- `bin/jim_path:139-168` — awk parser for `.jim/config.md` frontmatter; strips outermost `"..."` if present; whitespace-trims unquoted values.
- `bin/jim_path:183-188` — output: `printf '%s\n' "$value"`. Empty configured value → single newline → `$()` substitution yields empty string → `[ -z "$VAR" ]` is the disabled-check idiom for Bash callers.

**Build skill (sites that change)**
- `skills/build/SKILL.md:30-33` — step 1 resolve-config preamble reference (no change; only cite).
- `skills/build/SKILL.md:72-76` — Commit phase; L73 hardcodes `./pre-commit.sh`.
- `skills/build/SKILL.md:95-103` — step 6 Completion gate; new pre-completion step prepends here.

**Coder agent**
- `agents/coder.md:82` — Constraints bullet referencing `./pre-commit.sh`.

**Additional touch sites discovered (not in spec acceptance criteria)**
- `WORKFLOW.md:400` — `**Gate:** All tests pass. \`./pre-commit.sh\` is green.` — canonical SDLC process doc.
- `skills/debug/SKILL.md:81` — Scope Discipline bullet: `Does NOT run \`./pre-commit.sh\` or execute tests beyond what is needed to confirm reproduction.`

**Bash-placeholder pattern reference (no live consumers yet)**
- `skills/meta-test/SKILL.md:218-224` — canonical `$({jim_path} <key>)` invocation example. Spec 015 will be the first live consumer.
- `skills/_shared/resolve-paths.md:47-56` — `{jim_path}` derived-placeholder expansion (`jim_path --root='<abs>'`).

**Audit invariants the new code must satisfy**
- `skills/meta-test/SKILL.md:71-117` — Check 1 (preamble invocation).
- `skills/meta-test/SKILL.md:119-140` — Check 2 (schema value rules; new `string` keys must declare type explicitly).
- `skills/meta-test/SKILL.md:142-199` — Check 3 (no literal default in tool-arg position).
- `skills/meta-test/SKILL.md:201-242` — Check 4 (Bash uses `$({jim_path} <key>)` form).

## Local Patterns

**Schema additions** — Each prior addition (specs 011, 012, 013) appended `keys:` entries with one of five types and a per-section heading documenting purpose. Hook keys mirror this; no schema-shape changes.

**Bash placeholder composition** — No skill today uses `$({jim_path} <key>)`; spec 015 introduces the first invocations. The canonical idiom for an *optional* command (skip-if-empty) does not yet exist in the codebase. Two plausible forms for the planner:
```
HOOK="$($({jim_path}) hooks.pre-commit)"
[ -n "$HOOK" ] && bash -c "$HOOK"
```
or a one-liner with `&&` chains. Plan-level decision.

**Existing test framework** — Jim has no automated test suite. `bin/jim_path`'s contract is exercised via the inline verify battery in `docs/specs/jim/013-jim-path-helper/plan.md` Task 2 (mktemp fixture trees). The 015 plan should mirror this pattern: construct fixture `.jim/config.md` files with empty / non-empty `hooks.*` values and assert `jim_path` output. No new test framework needed.

**Quoting precedent for string-typed defaults** — `specs.id-prefix: ""` (config-schema.md:37). Empty-string default works under restricted YAML. Non-empty `string` defaults exist nowhere yet; the schema format constraint allows them only as unquoted plain scalars or quoted empty strings. Hook keys default to `""` so this is moot for defaults — but configured values may be quoted (`"make test"`) or unquoted (`make test`); the awk parser handles both.

## Security & Performance

**New shell-execution surface, same trust model.** The current hardcoded `./pre-commit.sh` already executes attacker-controlled shell on every build commit if the repo ships a malicious script. Replacing the literal with a configurable key in `.jim/config.md` does not widen that surface — the supply-chain risk noted in `ARCHITECTURE.md:234` already covers a malicious committed config. The `string` type's "any string accepted" rule defers trust to the user, matching the existing model.

**Embedded-quote limitation.** `bin/jim_path:160-162` strips only the outermost `"..."`; embedded escaped quotes are preserved literally and re-fed to `bash -c`, which double-interprets them. Users needing complex shell logic (pipes, embedded quotes, multi-line) should package it as a script file (`./ci.sh`) and configure `hooks.pre-completion: ./ci.sh`. The schema's "Hook keys" section should document this.

**Per-commit overhead.** Each TDD task produces 1–3 commits; each commit invokes `jim_path` once before the hook. ~ms overhead. Completion-gate invocation runs once per build. Negligible.

**Halting failure of pre-completion.** Acceptance criterion specifies halt before `/jim:arch` and `/jim:backlog`. Re-invoking `/jim:build` re-enters step 6 because all tasks are already `[x]` — verified against the build skill's step 6 logic (`skills/build/SKILL.md:95-103`).

## Recommendations

**1. Expand acceptance criteria to cover all touch sites.** Add `WORKFLOW.md:400` (rephrase the gate description) and `skills/debug/SKILL.md:81` (generalize the scope-discipline bullet, e.g., "Does NOT run the configured pre-commit hook or execute tests beyond what is needed to confirm reproduction" — or remove the literal example entirely). The spec's current `/jim:meta-test` audit criterion does not cover `WORKFLOW.md` or `skills/debug/SKILL.md` (meta-test scope is `skills/*/SKILL.md` and `agents/*.md` only — confirmed by `skills/meta-test/SKILL.md` audit-surface globs). A residual literal in `WORKFLOW.md` would not fail the audit but would violate the spirit of the spec.

**2. Plan should specify the empty-value Bash idiom.** Pick one form (`[ -n "$HOOK" ] && bash -c "$HOOK"` or equivalent) and document it in the build skill's Commit phase prose. Consistency matters because this is the first live `$({jim_path} <key>)` invocation and other skills will mirror it.

**3. Schema "Hook keys" section should include a usage note** about embedded-quote limits and the script-file workaround. Prevents future bug reports of the form "my config.md set `hooks.pre-commit: \"bash -c 'a && b'\"` and it didn't work."

**4. Consider whether `skills/debug/SKILL.md:81` should remove the literal entirely** rather than generalize. The scope-discipline bullet's intent is "debug doesn't run side effects beyond minimum reproduction"; naming the specific script was always incidental. Removing the example tightens the prose. PM call.

**Alignment statement:** Spec 015 aligns with `VISION.md:51-58` (Phase 3 — "Configuration and Integration") and `ARCHITECTURE.md:284-293` (Configuration and Overlay subsection of Plugin Conventions, including the `{jim_path}` Bash-invocation pattern). The new schema keys flow through the schema-is-authority invariant (`ARCHITECTURE.md:237`) and respect the four `/jim:meta-test` audit checks. No divergence from locked constraints. Phase 1 (External Intelligence) skipped per DoD: spec is internal-infra-only with no external API/library dependencies.

## Peer Feedback

**For PM:**
- **Spec 015 acceptance criteria miss two touch sites.** `WORKFLOW.md:400` and `skills/debug/SKILL.md:81` both reference `./pre-commit.sh` literally. The spec's "no literal `./pre-commit.sh` appears anywhere in the skill body" criterion covers `skills/build/SKILL.md`; the "no literal `./pre-commit.sh` appears" criterion covers `agents/coder.md`. Neither covers `WORKFLOW.md` (canonical SDLC doc) or `skills/debug/SKILL.md` (scope-discipline narrative). Recommend adding two acceptance criteria before approval:
  - `WORKFLOW.md` no longer references `./pre-commit.sh` literally; the build-section gate description is rephrased to reflect the configured pre-commit hook.
  - `skills/debug/SKILL.md` Scope Discipline bullet is generalized or the literal example is removed.
- **Note that `/jim:meta-test` will not catch a residual `WORKFLOW.md` or `skills/debug/SKILL.md` literal**, because both are outside the audit surface (`skills/*/SKILL.md` covers `debug` SKILL.md but the meta-test checks are designed for tool-call positions, not narrative scope-discipline bullets). Reviewer discipline is the only safeguard there.
