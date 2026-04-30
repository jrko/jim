---
title: "Configurable build hooks (pre-commit and pre-completion)"
type: feature
group: "jim"
id: "015"
status: approved
origin:
  - BACKLOG.md
  - docs/brainstorms/20260422-config-adherence.md
---

# 015 Configurable build hooks (pre-commit and pre-completion)

## Overview

Replace `/jim:build`'s hardcoded `./pre-commit.sh` invocation with two configurable hooks under a new `hooks.*` schema namespace, and add a completion-gate hook that runs before `/jim:arch` and `/jim:backlog` for fuller test runs too expensive to fire per-commit.

## Problem Statement

`/jim:build` today hardcodes `./pre-commit.sh` as the per-commit quality gate, which means projects without a script at that exact path cannot run jim out of the box — surfaced as a hard build failure during the 012-config-adherence dogfood. The hardcoding contradicts jim's broader config-adherence stance: every other path-shaped value in skill prose is a `{...}` placeholder resolved against `.jim/config.md`. Beyond that, projects with a slower CI suite (integration tests, end-to-end, lint matrices) have no place to run it: per-commit is too expensive, and there's no completion-gate equivalent. Today the only signal a `/jim:build` run produces is "all tasks marked `[x]`" — which says nothing about whether the cumulative change passes a fuller suite. Forking jim or post-build manual invocation are the only workarounds.

## User Stories

- As a jim user with a fast per-commit gate (e.g., `make test`, `./pre-commit.sh`, language-specific linters), I can configure `hooks.pre-commit` in `.jim/config.md` so that `/jim:build` honors my project's check command without forking jim.
- As a jim user with a slower full suite (integration tests, end-to-end, expensive lint matrix), I can configure `hooks.pre-completion` so that the suite runs once at the end of `/jim:build` before `ARCHITECTURE.md` and `BACKLOG.md` are regenerated, catching issues that were too expensive to run per-commit.
- As a jim contributor working on jim itself (no `./pre-commit.sh` script), I can run `/jim:build` without configuring any hook so that the empty-default disables the gate entirely and the build proceeds.
- As a jim user, I can leave either hook empty in `.jim/config.md` (or omit it) so that the corresponding gate is skipped without ceremony — opt-in, not opt-out.

## Acceptance Criteria

- [ ] `skills/_shared/config-schema.md` declares two new keys: `hooks.pre-commit` (type `string`, default `""`) and `hooks.pre-completion` (type `string`, default `""`).
- [ ] A new "Hook keys" section in `config-schema.md` documents each key's purpose, when it fires, and empty-disabled semantics — including an explicit note that the empty default provides no automated quality signal and that projects opt in by configuring a value.
- [ ] The Hook keys section documents the embedded-quote limitation: complex shell logic with pipes or nested quotes should be packaged as a script file and the script path configured as the hook value.
- [ ] The Hook keys section documents the trust model: `hooks.*` values are arbitrary shell commands executed by Bash; `.jim/config.md` is therefore a shell-execution authority equivalent to a committed script file; PR reviewers must scrutinize `hooks.*` changes the same way they scrutinize script-file content.
- [ ] The Hook keys section documents agent-context exposure: hook stdout and stderr are captured into the agent's conversation context — avoid hook commands that echo untrusted file contents, environment variables, or other attacker-controllable bytes, since those bytes flow into the agent's subsequent decisions.
- [ ] The spec's completion-step `/jim:arch` differential update reflects the new shell-execution surface introduced by `hooks.*` — `ARCHITECTURE.md`'s Security Considerations section names `.jim/config.md` as a shell-execution authority alongside the existing path-tampering mitigations.
- [ ] `skills/build/SKILL.md` Commit phase invokes the resolved `hooks.pre-commit` value via Bash before each TDD commit; no literal `./pre-commit.sh` appears anywhere in the skill body.
- [ ] `skills/build/SKILL.md` step 6 (Completion gate) prepends a hook step that invokes the resolved `hooks.pre-completion` value via Bash before `/jim:arch`; the new step appears before the existing `/jim:arch` and `/jim:backlog` invocations.
- [ ] `agents/coder.md` Constraints section no longer references `./pre-commit.sh` literally — the wording is generalized to "the configured pre-commit hook" or equivalent.
- [ ] `WORKFLOW.md` no longer references `./pre-commit.sh` literally; the `/jim:build` section's gate description is rephrased to reflect the configured pre-commit hook (e.g., "the configured `hooks.pre-commit` is green, when set").
- [ ] `skills/debug/SKILL.md` Scope Discipline bullet at L81 is generalized or the literal `./pre-commit.sh` example is removed — the bullet's intent (debug runs no side-effect tests beyond minimum reproduction) is preserved.
- [ ] When the resolved `hooks.pre-commit` value is the empty string, the Commit phase skips the hook step entirely — no Bash invocation, no error, commit proceeds.
- [ ] When the resolved `hooks.pre-commit` value is non-empty, the configured command runs via Bash before every commit; on non-zero exit the build does not commit and retries the hook until it passes (unbounded — unchanged from current `./pre-commit.sh` behavior).
- [ ] When the resolved `hooks.pre-completion` value is the empty string, step 6 proceeds directly to `/jim:arch` — no Bash invocation, no error.
- [ ] When the resolved `hooks.pre-completion` value is non-empty, the configured command runs via Bash at step 6 entry; on non-zero exit `/jim:build` halts the completion gate immediately — `/jim:arch` is not invoked, `/jim:backlog` is not invoked, the "mark plan complete?" prompt is not shown, the failure output is reported, and the user can re-invoke `/jim:build` (which re-enters step 6 since all tasks are already `[x]`).
- [ ] Multi-token commands (e.g., `make test`, `bash -c "a && b"`) work via the existing schema string-quoting rules — no array syntax introduced.
- [ ] `/jim:meta-test` audit passes after the changes — `{hooks.pre-commit}` and `{hooks.pre-completion}` placeholders never flow into tool calls unresolved, and no literal default value leaks into tool-argument positions in skill or agent prose.

## Out of Scope

- Additional hook event types (e.g., `hooks.post-commit`, `hooks.pre-task`, `hooks.pre-build`, `hooks.post-completion`). Introduce when concrete need surfaces, not preemptively.
- A new schema value type for shell commands (e.g., `shell-command` with stricter validation). Existing `string` type is sufficient — the trust model matches the user's shell.
- Array or list syntax for chaining multiple commands per hook. Users can chain via shell (`bash -c "a && b"`).
- Hook execution timeouts, sandboxing, or resource limits. Same trust model as today.
- A retry cap on `hooks.pre-commit` failures or automatic `/jim:debug` invocation when the hook fails repeatedly. Behavior is unbounded retry until pass — unchanged from today's `./pre-commit.sh` flow.
- A backward-compatibility default of `./pre-commit.sh` for `hooks.pre-commit`. Rejected: jim itself has no such script, and no project currently depends on the hardcoded literal.
- Hook configuration for `/jim:debug` or any skill other than `/jim:build`. This spec is `/jim:build`-only.
- A migration path or warning for users who have existing `./pre-commit.sh` scripts. Configuring `hooks.pre-commit: "./pre-commit.sh"` in `.jim/config.md` restores prior behavior — documented in the schema's Hook keys section.

## Open Questions

None — all interview questions resolved.
