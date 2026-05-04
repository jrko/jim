---
spec: "spec.md"
status: Needs Plan Review
date: "2026-05-03"
---

# Security Review: bin/jim_run_hook — single-call hook dispatcher

## Summary

Spec-scoped review of the proposed `bin/jim_run_hook` helper that resolves and dispatches configured `hooks.<event>` shell commands. Initial spec-phase review surfaced two Notable findings (F1 quoting discipline, F2 ARCHITECTURE Security Considerations) and two Advisory findings (F3 sysexits exit codes, F4 empty-event handling). F1 and F2 were routed back into the spec (now AC L32 and L42, resolved). The follow-on plan-phase review surfaces one new Notable finding (F5 — jim_path-failure vs hook-failure prose conflation in Task 4). F3 remains Advisory/backlog; F4 was routed to Plan and the planner deferred to backlog with rationale (accepted).

**Artifacts reviewed:**
- `docs/specs/jim/016-jim-run-hook-helper/spec.md` (updated with F1, F2 amendments)
- `docs/specs/jim/016-jim-run-hook-helper/research.md`
- `docs/specs/jim/016-jim-run-hook-helper/plan.md` (new — added in differential review)

## Findings

### 1. Helper-internal argument quoting is load-bearing for shell-injection safety, but not specified — RESOLVED

- **Severity:** Notable (resolved)
- **Status:** Resolved by spec amendment (AC L32) and plan implementation (Decision 2; Task 1 verify cases 7 and 7b cover the metacharacter-injection test).
- **Description:** The helper's first action is to invoke `jim_path --root='<abs>' hooks.<event>` to resolve the configured value. The `<event>` argument arrives from the caller (the agent via `{jim_run_hook}` substitution) and is concatenated into a `hooks.<event>` key string passed to `jim_path`. If the helper's internal invocation expands `$event` unquoted (e.g., `value=$(jim_path --root=$root hooks.$event)`), shell metacharacters in `<event>` execute *before* `jim_path` ever validates the key — `jim_path`'s schema-driven allowlist (interview Q2's load-bearing protection) is bypassed entirely. The same risk applies to `$root`. Spec 015's trust model permits arbitrary commands in *configured* values; it does not permit shell injection through *helper-CLI args*.
- **Suggestion:** ~~Add an AC mandating quoting discipline and a verify-battery test case with shell metacharacters in `<event>`.~~ Done — spec AC L32 added; plan Task 1 implements via `"hooks.$event"` and `--root="$root"`; verify cases 7 and 7b assert no metacharacter shell-evaluation against canary file.
- **Route:** ~~Spec~~ Resolved

### 2. Helper is a load-bearing component of the shell-execution authority but not named in ARCHITECTURE.md Security Considerations — RESOLVED

- **Severity:** Notable (resolved)
- **Status:** Resolved by spec amendment (AC L42) and plan task (Task 8 explicitly extends the Security Considerations paragraph).
- **Description:** ARCHITECTURE.md L237's Security Considerations paragraph names `skills/_shared/config-schema.md`, `skills/_shared/resolve-paths.md`, and `bin/jim_path` as the trio load-bearing for config-mediated filesystem safety. Spec 015 added a "Shell-execution authority" paragraph naming `hooks.*` values as arbitrary shell commands, with the build-skill consumer pairing `$({jim_path} hooks.X)` substitution with `||`-chain exit-code capture for fail-loud posture. Spec 016 makes `bin/jim_run_hook` the new sole consumer of that capture: a buggy helper that silently catches `jim_path` failures or skips the empty-check would silently disable the configured gate without any signal.
- **Suggestion:** ~~Add an AC for ARCHITECTURE Security Considerations update.~~ Done — spec AC L42 added; plan Task 8 implements with prose naming the helper as load-bearing alongside `bin/jim_path` and citing the fail-loud / skip-only-on-empty invariants as runtime enforcement.
- **Route:** ~~Spec~~ Resolved

### 3. Exit-code collision between helper-internal failures and hook-reported exit codes

- **Severity:** Advisory
- **Description:** The helper propagates two classes of exit codes through the same channel: (a) `jim_path`'s exit codes (1 = other error, 2 = unknown key) when `jim_path` itself fails, and (b) the configured hook's exit code when the hook runs. A hook that legitimately exits 2 (e.g., a linter signaling a specific failure class — `shellcheck` returns 2 for syntax errors, `jq` returns 2 for invalid input) becomes indistinguishable from "unknown key" from the agent's perspective. The agent's downstream `/jim:debug` invocation may misroute as a result. No security impact today, but a hardening opportunity for future debuggability.
- **Suggestion:** Use the sysexits.h convention for helper-internal failures: exit 70 (EX_SOFTWARE) when `jim_path` errors with code 1, exit 78 (EX_CONFIG) when `jim_path` errors with code 2 (unknown key — likely indicates schema/config mismatch). Reserve exit codes 0-63 for passthrough of hook-reported codes. Add to backlog with a one-line note in the helper's stderr explaining the mapping.
- **Route:** Backlog

### 4. Empty-event-arg handling unspecified — DEFERRED

- **Severity:** Advisory (deferred to backlog by planner)
- **Status:** Routed to Plan in initial review; planner declined inclusion (plan Out of Scope: "defensive nicety, not a security requirement; deferred to backlog or to a follow-on spec if user feedback warrants"). Deferral accepted — no security-critical regression results from the helper inheriting jim_path's empty-key behavior.
- **Description:** Spec does not specify what happens when `jim_run_hook` is invoked with no `<event>` argument. Helper's behavior defaults to whatever `jim_path` emits when given no key (likely the `missing key` error). Defensive coding suggests an explicit usage error and a distinct exit code.
- **Suggestion:** ~~Planner adds explicit usage check.~~ Deferred to backlog — see plan Out of Scope.
- **Route:** ~~Plan~~ Backlog (via planner deferral)

### 5. Task 4's Commit-phase prose conflates jim_path-failure with hook-failure (loses recoverable/unrecoverable distinction)

- **Severity:** Notable
- **Description:** Plan Task 4 replaces `skills/build/SKILL.md` Commit phase (L72-78) with single-line `{jim_run_hook} pre-commit` plus prose: "...exits non-zero (fail-loud) on resolution failure. If the hook exits non-zero: show the error output, fix the issues, re-run all tests, and re-run the hook until it passes. Do NOT commit until the hook is green." This conflates two distinct failure modes that spec 015 Decision 5 explicitly distinguished: (a) **jim_path resolution failure** (helper exits 1 or 2 with stderr prefixed `jim_path:`) — unrecoverable in-loop; user must fix `.jim/config.md`; retrying the hook will keep failing the same way; and (b) **hook execution failure** (helper exits non-zero with hook stderr) — recoverable; standard fix-test-rerun loop. The new prose's retry instruction applies the recoverable loop unconditionally. If a user introduces a typo in `.jim/config.md` mid-build, the agent would re-run the hook indefinitely, never surfacing the underlying config issue — silently disabling the user's ability to commit and burning iteration cycles. Security impact: degrades the fail-loud signal that spec 015 Decision 5 was designed to preserve, even though the helper itself fails-loud correctly. The signal is lost in skill-prose translation.
- **Suggestion:** Amend Plan Task 4's proposed replacement prose to distinguish the two failure modes explicitly. Suggested wording: "Run `{jim_run_hook} pre-commit` via Bash before committing. The helper resolves `hooks.pre-commit` from `.jim/config.md`, skips silently if unset, and runs the configured command otherwise. **Two failure modes:** (a) If stderr starts with `jim_path:` (resolution failure — malformed config, schema-read error, plugin not loaded), STOP. Do NOT commit. Surface the message to the user — this is a `.jim/config.md` problem, not a code problem. (b) If the configured hook itself exits non-zero (hook stderr without the `jim_path:` prefix): show the error output, fix the issues, re-run all tests, and re-run the hook until it passes. Do NOT commit until the hook is green." Task 5 (Completion gate) already STOPs on any non-zero exit so it does not need amendment.
- **Route:** Plan

## STRIDE Coverage

| Category | Relevant? | Findings |
| :--- | :--- | :--- |
| Spoofing | N/A | PATH hijacking of `jim_path` from a malicious shell environment is a pre-existing risk for any helper that calls another helper; not introduced by spec 016. No new authentication surface. |
| Tampering | Yes | Configured `hooks.*` values are arbitrary shell commands by trust-model design (spec 015). Helper does not validate value content. F1 and F2 (resolved) cover the helper's role in preserving the fail-loud signal that detects config-tampering at runtime. F5 (new) covers the skill-prose layer that surfaces that signal — if the prose conflates failure modes, the runtime fail-loud signal is degraded by retry-loop masking. |
| Repudiation | N/A | No audit-trail expectation. The configured hook value in `.jim/config.md` is the audit trail (committable, PR-reviewable). No change from spec 015's posture. |
| Information Disclosure | No | Helper passes through `jim_path`'s stderr (already constrained per `bin/jim_path:180` to one-line non-content errors) and the configured hook's stderr (already documented as agent-context-exposed in spec 015). No new exposure. |
| Denial of Service | No | Configured hooks have no timeout — explicitly out-of-scope per spec 015 trust model. Helper inherits this; no new DoS surface. A pathological hook (`sleep 999999`) hangs `/jim:build` today and continues to do so under spec 016. |
| Elevation of Privilege | N/A | Helper runs with the invoking user's privileges. No suid, no privilege boundary, no setuid bit on `bin/jim_run_hook`. Same posture as `bin/jim_path`. |

## Routing Recommendations

### Resolved (no further action)

- **F1:** Spec AC L32 + Plan Decision 2 + Plan Task 1 verify cases 7/7b.
- **F2:** Spec AC L42 + Plan Task 8.

### Plan amendments (open)

- **F5:** Amend Plan Task 4's replacement prose to distinguish jim_path-failure (STOP, no retry) from hook-failure (retry-until-green). Suggested wording in F5 above.

### Backlog items (open)

- **F3:** Helper exit-code disambiguation via sysexits ranges (70 for `jim_path` other-error, 78 for unknown-key, 64 for usage; reserve 0-63 for hook passthrough).
- **F4:** Empty-`<event>`-arg explicit usage error (planner-deferred from Plan to Backlog).
