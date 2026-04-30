---
spec: "spec.md"
status: Needs Plan Review
date: "2026-04-29"
---

# Security Review: Configurable build hooks (pre-commit and pre-completion)

## Summary

Spec 015 elevates `.jim/config.md` from a path/identifier configuration surface to a **shell-execution authority**: the new `hooks.pre-commit` and `hooks.pre-completion` keys are arbitrary shell strings passed to Bash during `/jim:build`. The trust model is unchanged from today's hardcoded `./pre-commit.sh` (both execute committed shell as the invoking user), but the *surface* shifts from a script file to a config-file line, with implications for PR-review scrutiny and agent-context exposure. The plan now adds a concrete Bash invocation idiom that introduces a fresh fail-open concern: jim_path errors silently disable the hook. No critical findings; one notable spec-phase item (now resolved at spec layer), one notable plan-phase item (fail-open idiom), three advisories.

**Artifacts reviewed:** `docs/specs/jim/015-build-hooks/spec.md`, `docs/specs/jim/015-build-hooks/research.md`, `docs/specs/jim/015-build-hooks/plan.md`. Findings 1-4 are spec-phase (Findings 1, 2, 4 resolved at spec layer; Finding 3 routed to backlog). Finding 5 is plan-phase.

## Findings

### 1. `hooks.*` shifts shell-execution surface from script file to config line — PR-review attention may not match

- **Severity:** Notable
- **Description:** After 015, a one-line change to `.jim/config.md` (e.g., `hooks.pre-commit: rm -rf $HOME`) executes on the next `/jim:build` invocation. Same trust model as today's `./pre-commit.sh` (both rely on the developer reading what they're about to run), but PR reviewers tend to scrutinize script-file diffs more carefully than config-file diffs — the "obvious shell" cue moves into a less visible surface. The schema's `string` type rule ("any string accepted") defers all validation to the user, and `ARCHITECTURE.md:234`'s existing supply-chain mitigation (path-relative, no `..`, realpath containment) covers path-typed values, not string-typed shell commands. This is a real gap in the existing trust documentation, not a new vulnerability — but the spec is the natural place to close it.
- **Suggestion:** Expand the spec's existing "Hook keys schema-section" acceptance criterion to require explicit prose stating: (a) `hooks.*` values are arbitrary shell commands executed by Bash; (b) `.jim/config.md` is therefore a shell-execution authority equivalent to a committed script file; (c) PR reviewers must scrutinize `hooks.*` changes the same way they scrutinize script-file content. Add a parallel acceptance criterion that ARCHITECTURE.md's Security Considerations section is updated during the spec's `/jim:arch` differential-update completion step to reflect this shifted surface.
- **Route:** Spec

### 2. Hook stdout/stderr flows into agent context — prompt-injection-via-hook-output is possible

- **Severity:** Advisory
- **Description:** Hook commands run via the Bash tool, which surfaces stdout/stderr into the agent's conversation context. A hook that echoes attacker-controlled content (e.g., `cat untrusted-file && exit 0`) can inject prompt-injection-style content at a sensitive moment — particularly the new `hooks.pre-completion`, which fires immediately before `/jim:arch` regenerates `ARCHITECTURE.md` and `/jim:backlog` regenerates `BACKLOG.md`. Same risk model as today's `./pre-commit.sh` output, but the pre-completion timing is fresh and worth flagging.
- **Suggestion:** Add a documentation-only line to the schema's "Hook keys" section: "Hook stdout and stderr are captured into the agent's conversation context. Avoid hook commands that echo untrusted file contents, environment variables, or other attacker-controllable bytes — those bytes flow into the agent's subsequent decisions." Folds into the Finding 1 expanded acceptance criterion; no separate criterion needed.
- **Route:** Spec

### 3. Pre-completion hook can side-effect the doc-regen flow even on success

- **Severity:** Advisory
- **Description:** The pre-completion hook runs at step 6 entry, before `/jim:arch` scans the codebase and before `/jim:backlog` regenerates the deferred-work index. A *successful* hook (exit 0) that side-effects project files between hook return and `/jim:arch` can subtly corrupt the differential update — e.g., a hook that modifies plan.md to mark unchecked tasks complete, or one that touches ARCHITECTURE.md to bias the architect's diff. The spec's failure semantics protect against non-zero exits, not against successful-but-side-effecting runs. This is the same trust model as any committed shell script, but the timing window between pre-completion and the doc-regen calls is a new attack moment that didn't exist before this spec.
- **Suggestion:** Add a backlog item: "Consider read-only invariant check between `hooks.pre-completion` success and `/jim:arch` invocation — record the mtime/hash of `plan.md`, `ARCHITECTURE.md`, and `spec.md` before the hook runs and verify they're unchanged after. Out of scope for 015 (defers to user trust per the existing model), but worth tracking if dogfooding surfaces an incident." No spec change required — design accepts the trust model.
- **Route:** Backlog

### 4. Empty default removes the prior fail-loud safety net for fresh projects

- **Severity:** Advisory
- **Description:** With `hooks.pre-commit` defaulting to `""` (disabled), a developer cloning a jim-using project and running `/jim:build` for the first time gets no automated quality enforcement until they configure a hook. Before 015, the hardcoded `./pre-commit.sh` literal at least produced a fail-loud "script not found" error — degraded UX, but not a silent absence-of-gate. The trust posture shifts from "fail-loud-without-script" to "silent-no-gate-by-default." Not a security vulnerability — but the spec already accepts the empty-default design (Out of Scope: "A backward-compatibility default of `./pre-commit.sh`"), so this finding is hardening-via-documentation rather than a design challenge.
- **Suggestion:** When `/jim:config` is later updated to scaffold the new keys (a follow-on concern, not part of 015 per its scope), the scaffolding prompt should explicitly ask the user whether they want to wire a `hooks.pre-commit` command, defaulting to a guided suggestion rather than empty. For 015 itself, the schema's "Hook keys" section should note the empty-default posture explicitly so users don't assume a hidden gate exists. Folds into the Finding 1 expanded acceptance criterion.
- **Route:** Spec

### 5. Hook invocation idiom is fail-open on jim_path errors

- **Severity:** Notable
- **Description:** The plan's Bash idiom — `HOOK="$({jim_path} hooks.pre-commit)"; if [ -n "$HOOK" ]; then bash -c "$HOOK"; fi` (Interface Contracts; Tasks 3 and 4) — captures only stdout via `$()` and ignores `bin/jim_path`'s exit code. If jim_path exits non-zero (malformed `.jim/config.md`, schema-read failure, plugin-root plausibility check failure per `bin/jim_path:22-27`, or `command not found` if the plugin is unloaded), `$HOOK` is the empty string and the hook silently skips — fail-open, not fail-loud. From a tampering perspective, this widens Finding 1's surface: an attacker who corrupts `.jim/config.md` to malformed YAML silently disables the pre-commit gate, allowing commits to proceed without quality enforcement. The same fail-open pattern exists in Task 1's verify (`[ "$(jim_path hooks.pre-commit)" = "" ]` passes spuriously if jim_path crashes). Spec 015 introduces the first live consumers of `$({jim_path} <key>)` substitution, so any pattern set here propagates to downstream skills that mirror the idiom — this is the canonical place to set fail-loud discipline.
- **Suggestion:** Update the Bash idiom in the plan's Interface Contracts (and Tasks 3, 4) to capture jim_path's exit code and halt loudly on failure. Two viable forms:

  ```
  if ! HOOK="$({jim_path} hooks.pre-commit)"; then
    echo "jim_path failed; aborting commit" >&2
    exit 1
  fi
  if [ -n "$HOOK" ]; then bash -c "$HOOK"; fi
  ```

  Or with `||` chain:

  ```
  HOOK="$({jim_path} hooks.pre-commit)" || { echo "jim_path failed" >&2; exit 1; }
  [ -n "$HOOK" ] && bash -c "$HOOK"
  ```

  Apply the same exit-code-aware pattern to Task 1's verify so it also halts on jim_path failure (e.g., chain the two `jim_path` calls with `&&` so non-zero from either propagates). Add a fifth Design Decision block to the plan documenting the fail-loud posture explicitly — future skills will mirror this pattern, and the rationale should travel with it.
- **Route:** Plan

## STRIDE Coverage

| Category | Relevant? | Findings |
| :--- | :--- | :--- |
| Spoofing | N/A | Jim has no identity model. Shell commands run as the invoking user with no impersonation surface. |
| Tampering | Yes | Finding 1 — `.jim/config.md` is the primary tampering vector for shell-execution surface. Finding 5 — fail-open idiom widens the tampering surface: malformed config silently disables the hook. |
| Repudiation | N/A | Jim has no audit log; hook executions are visible in the agent transcript. Existing posture, no change. |
| Information Disclosure | Yes | Finding 2 — hook stdout/stderr flows into agent context, enabling prompt-injection-via-hook-output. |
| Denial of Service | No | A long-running hook hangs `/jim:build`, but no remote attacker is in the threat model. Same as any committed shell script today. No new finding. |
| Elevation of Privilege | N/A | Hook runs with the user's shell privileges. No new privilege boundary introduced. |

## Routing Recommendations

### Spec amendments *(resolved)*
- **Finding 1 (Notable):** Expand the existing Hook keys schema-section acceptance criterion to require trust-model documentation (shell-execution authority, PR-review parity with script files). Add a parallel criterion for ARCHITECTURE.md's Security Considerations update at `/jim:arch` completion. *Resolved — spec criteria 4 and 6 added; plan Tasks 2 and 8 implement.*
- **Finding 2 (Advisory):** Add a documentation-only line to the Hook keys section about agent-context exposure of hook output. *Resolved — spec criterion 5; plan Task 2 implements.*
- **Finding 4 (Advisory):** Document the empty-default posture in the Hook keys section. *Resolved — spec criterion 2; plan Task 2 implements.*

### Plan amendments
- **Finding 5 (Notable):** Update the Bash invocation idiom to capture jim_path's exit code and halt loudly on failure. Apply the same fix to Task 1's verify command. Add a Design Decision block documenting the fail-loud posture so downstream skills mirror it. The plan is the first live consumer of `$({jim_path} <key>)` substitution, making it the canonical place to set the pattern.

### Backlog items *(routed)*
- **Finding 3 (Advisory):** Read-only invariant check between `hooks.pre-completion` success and `/jim:arch` invocation. File-mtime/hash verification of `plan.md`, `ARCHITECTURE.md`, `spec.md`. *Routed — added to BACKLOG.md ad-hoc section as "Read-only invariant check for hooks.pre-completion side effects".*
