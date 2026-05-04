---
spec: "spec.md"
status: Active
date: "2026-05-03"
---

# Research: bin/jim_run_hook — single-call hook dispatcher

## Anchors

**Helper template (mirror this structure):**

- `bin/jim_path:1` — `#!/usr/bin/env bash` shebang. Matches user CLAUDE.md rule and jim precedent. New helper uses identical shebang.
- `bin/jim_path:5` — `set -euo pipefail`. New helper inherits the same fail-fast posture.
- `bin/jim_path:7-15` — portable `BASH_SOURCE` symlink resolution loop (no `readlink -f` — GNU-only). New helper does NOT need this if it never reads its own location; it can rely on PATH lookup of `jim_path`. Decision noted in Recommendations.
- `bin/jim_path:17-19` — plugin root derivation via `dirname` twice + `cd -P && pwd -P`.
- `bin/jim_path:22-27` — plausibility check against `<plugin_root>/.claude-plugin/plugin.json` containing `"name": "jim"`. New helper inherits this protection transitively via `jim_path` invocation; no duplicate check needed (see Recommendations).
- `bin/jim_path:29-65` — argument parsing: `--root <abs>` flag + positional key. New helper parses `--root` identically and forwards it; positional becomes `<event>` followed by hook args.
- `bin/jim_path:183-188` — output via `printf '%s\n' "$value"`; exit 0 success, 2 unknown-key, 1 other-error.

**Consumer to rewrite:**

- `skills/build/SKILL.md:72-78` — Commit phase bash idiom (the `**Commit**` bullet's bash block plus prose). Replace entirely with `{jim_run_hook} pre-commit` invocation.
- `skills/build/SKILL.md:106-111` — Completion gate step 6, sub-step 1 bash idiom. Replace with `{jim_run_hook} pre-completion`.

**Schema and preamble:**

- `skills/_shared/config-schema.md:111-119` — Derived Placeholders table. New row for `{jim_run_hook}` parallel to `{jim_path}`.
- `skills/_shared/config-schema.md:96-109` — Hook keys section. Add a one-line note pointing to the helper as canonical invoker; keep trust-model, embedded-quote, and agent-context-exposure paragraphs intact (they describe the configured value's properties, not the invocation mechanism).
- `skills/_shared/resolve-paths.md:47-57` — Step 4 derived-placeholder definition. Add `{jim_run_hook}` source/expansion-form/quoting-algorithm/halt-condition prose, mirroring the existing `{jim_path}` paragraph.

**Architecture doc:**

- `ARCHITECTURE.md:172-180` — Plugin Executables `bin/jim_path` entry. Add a parallel `bin/jim_run_hook` entry directly below.
- `ARCHITECTURE.md:284-291` — Configuration and Overlay section. The "Build hooks" bullet (currently L291) describes the in-prose idiom; rewrite to describe the helper as the dispatcher.

**Audit gate:**

- `skills/meta-test/SKILL.md:74` (Check 1 — preamble), `:143-150` (Check 3 — literal defaults), `:151-180` (Check 4 — `$({jim_path} <key>)` substitution). The new helper passes Check 1 (no new preambles), Check 3 (removes idiom prose, leaves no literal defaults), and Check 4 (skill prose uses `{jim_run_hook}` placeholder, never literal `jim_run_hook --root='...'`). See Recommendations on whether Check 4 needs to be generalized.

**Verify-battery template:**

- `docs/specs/jim/013-jim-path-helper/plan.md:248-273` (Task 2) — verify-battery pattern: `mktemp -d` fixture root, `mkdir` plugin structure, fixture `plugin.json` and config, then test cases with `test $? -eq N` assertions and `grep` on stderr. New helper's plan should mirror this exact shape.

## Local Patterns

**Helper conventions** (from `bin/jim_path`):
- Errors prefix with `jim_path: ` on stderr, then exit non-zero.
- Stdout reserved for the resolved value; stderr for diagnostics.
- One-line error messages — no markdown, no multi-line stacktraces. Stderr flows into agent context, so terseness is a security property (`ARCHITECTURE.md:180` Key Constraints).
- No external library dependencies; only `awk`, `grep`, `dirname`, `cd`, `pwd`, `printf` — POSIX baseline.

**Test discipline** (from `013-jim-path-helper/plan.md:248-273` and `012-config-adherence/plan.md:84`):
- Jim has no automated test framework. Verify commands ARE the tests.
- Pattern: `mktemp -d` isolated fixture root → minimal seed config/manifest → shell assertions (`test`, `grep`, `diff`) → cleanup via shell `trap` or implicit on exit.
- Assertions exit non-zero on failure; the verify command's exit code is the signal.

**Derived placeholder convention** (from `skills/_shared/resolve-paths.md:47-57`):
- Multi-token expansion (helper-name + `--root='<abs>'`) is the documented pattern for any future helper that resolves to a shell command. Spec 016 is the first follow-on use, so the precedent it sets matters for future hook-event helpers or other dispatchers.

**Skill prose style** (from `skills/build/SKILL.md` and the global pattern):
- Bash blocks in prose stay short and analyzable. Spec 015 was the first to embed multi-line bash; the failure mode it surfaced (heuristic interruption) is the motivating evidence for moving the dispatch into a helper.

## Security & Performance

**Bash heuristic source — direct evidence (Claude Code 2.1.121):**

The Claude Code Bash tool's permission heuristic uses tree-sitter-bash AST node-type dispatch, with three sets governing acceptance:

- `EkK` (always-rejected, emits `Contains <type>`): `if_statement`, `for_statement`, `while_statement`, `until_statement`, `case_statement`, `compound_statement` (`{ ... }` brace groups), `subshell` (`( ... )`), `test_command` (`[[ ... ]]`), `function_definition`, `command_substitution` (in non-assignment positions), `process_substitution`, `expansion`, `simple_expansion`, `brace_expression`, `ansi_c_string`, `translated_string`, `herestring_redirect`, `heredoc_redirect`.
- `ok_` (accepted top-level wrappers): `program`, `list`, `pipeline`, `redirected_statement`.
- `ak_` (accepted statement separators): `&&`, `||`, `|`, `;`, `&`, `|&`, newline.
- `mW()` flag (strictness gate): when on, command-name slots reject `simple_expansion`/`expansion` directly and reject `string`/`concatenation` containing variable expansion via `xkK`. The `cM(H)` formatter emits `Unhandled node type: <type>` for nodes not in `EkK` — the source of the `Unhandled node type: string` failure observed during spec 015 dogfooding.

**Implication:** every in-prose form that resolves a string-typed config value and invokes it as a command trips at least one rejection. The helper-subprocess path bypasses the heuristic entirely because subprocess content is not analyzed — only the agent's top-level Bash invocation (`jim_run_hook pre-commit`) is.

**Trust model preservation:**
- Configured hook values remain arbitrary shell commands run with user privileges (spec 015 trust model). The helper does not validate value content.
- Hook stdout/stderr remain captured by the Bash tool into agent context (spec 015 agent-context-exposure invariant). The helper does not redirect or filter.
- The helper trusts `jim_path`'s plausibility check (`bin/jim_path:22-27`). A misinstalled plugin causes `jim_path` to exit 1; the helper's fail-loud propagates that. No duplicate check needed.
- Permission-prompt visibility tradeoff (acknowledged in spec): prompts now show `jim_run_hook pre-commit` instead of the literal hook command. Mitigation is helper readability — `bin/jim_run_hook` is small enough to PR-review once.

**Performance:** the helper adds one `exec` boundary (helper process → configured command). Negligible. Per-commit overhead is dominated by the hook itself.

## Recommendations

**1. Helper exec form** — three options for the resolve-then-invoke step inside the helper:

- A. `exec bash -c "$value \"\$@\"" bash "$@"` — fork-replace with `bash -c`, $0=`bash`, args propagate via `"$@"`. Supports shell metacharacters in the configured value (pipes, redirects, `&&`) which spec 015's embedded-quote note explicitly relies on.
- B. `eval "$value \"\$@\""` — runs in current helper shell. Same metacharacter support, but `eval` quoting is fragile for arg-forwarding.
- C. `exec $value "$@"` — direct exec. Loses shell metacharacters; configured value must be a single executable path.

**Recommend A.** Matches spec 015's "complex shell logic via `bash -c`" expectation; clean exit-code propagation via fork-replace; arg-forwarding is unambiguous.

**2. Plausibility check duplication** — the helper could re-derive its own plugin root and check `plugin.json`, mirroring `bin/jim_path:7-27`. Skip it. The first thing the helper does is invoke `jim_path`, which performs the same check; a misinstalled plugin causes `jim_path` to exit 1 and the helper's fail-loud surfaces it. Duplicating the check adds ~20 lines and a second symlink-resolution loop for no functional gain.

**3. Verify battery scope** (for the planner) — mirror `013-jim-path-helper/plan.md:248-273`. Cover these cases:
- Empty configured value → exit 0, no output, no command execution.
- Non-empty value, simple command → exit 0, command output passes through.
- Non-empty value with shell metacharacters (`&&`, pipes) → exit code propagates correctly.
- Forwarded positional args reach the configured command.
- `jim_path` failure (unknown key, malformed config) → helper exits non-zero, jim_path's stderr surfaces unchanged.
- `--root` flag forwarding (helper invoked from a subdirectory of the project).

**4. Meta-test Check 4 — needs generalization?** `skills/meta-test/SKILL.md:151-180` currently audits for `$({jim_path} <key>)` substitution patterns. The new `{jim_run_hook}` placeholder is a sibling pattern. Two options:
- A. Leave Check 4 as jim_path-specific. The new helper's prose calls (`{jim_run_hook} pre-commit`) are not `$()`-substituted, so they don't match the existing audit; they're checked by Check 3 (literal-defaults) instead. Acceptable.
- B. Generalize Check 4 to "any derived placeholder". Forward-looking but speculative.

**Recommend A.** Defer generalization until a third derived-placeholder helper lands. Single-instance generalization is premature.

**5. `<event>` validation** — helper does no validation; `jim_path hooks.<event>` returns exit 2 (`unknown key`) if the event isn't in the schema. Trust the schema. (Already settled in interview Q2.)

## Alignment

**VISION.md:** Spec aligns with Phase 1 (Core SDLC) — fixes a Phase 1 quality regression introduced by spec 015's in-prose idiom. Supports Phase 3 (Configuration and Integration) by stabilizing the hook-dispatch pattern that future hook events will reuse.

**ARCHITECTURE.md:**
- Schema-is-the-authority (`L237`, `L170`): respected — helper trusts `jim_path`'s schema-driven validation; no duplicate allowlist.
- Plugin Executables convention (`L172-180`): respected — new helper parallels `bin/jim_path` (bash, no library deps, plugin `bin/` PATH discovery, fail-loud stderr conventions).
- Resolve-before-tool-call (`L237`, `L286-289`): respected — new `{jim_run_hook}` placeholder follows the documented multi-token derived-placeholder pattern; never flows into tool calls unresolved.
- `_shared/` is plugin contract not overlayable (`L170`, `L292`): adding `{jim_run_hook}` to `resolve-paths.md` is a plugin-contract update, properly scoped through this spec.
- Anti-patterns (`L296-302`): no Personality Soup (helper has no persona); no Permission Creep (helper inherits jim_path's least-privilege posture); no Instruction Shadowing (no rules duplicated from elsewhere); no Duplicate Logic (validation lives in jim_path, not duplicated in helper).

No divergence from locked constraints.
