---
spec: "spec.md"
status: Active
date: "2026-04-26"
---

# Security Review: jim_path helper — shell-mediated config-adherent path resolution for Bash calls

## Summary

Spec-and-plan review of the `jim_path` helper, two passes complete. **Spec phase (findings 1–5):** three Notable routed to Spec — all addressed via amendments to `spec.md`; one Advisory (YAML parsing) was deferred to plan-phase and is addressed by `plan.md` Decision 3 + Contract B; one Advisory routed to `BACKLOG.md`. **Plan phase (findings 6–9):** one Notable (macOS `readlink -f` portability) and three Advisory (plugin-root plausibility check, awk-parser test coverage, PATH-precedence assumption) — all addressed via amendments to `plan.md` Contracts A and D, Decision 8, and Tasks 2 and 5. No open findings remain.

**Artifacts reviewed:**
- `docs/specs/jim/013-jim-path-helper/spec.md`
- `ARCHITECTURE.md`, `VISION.md`
- `skills/_shared/resolve-paths.md`, `skills/_shared/config-schema.md` (for context)
- `docs/specs/jim/012-config-adherence/spec.md` and `security.md` (precedent for findings 1 and 2)

## Findings

### 1. Helper trust contract is implicit — preamble bypass creates value-validation gap

- **Severity:** Notable
- **Status:** **Addressed.** `spec.md` Helper-script section now carries an explicit Trust contract acceptance criterion documenting the preamble-validates-first assumption and cross-referencing the BACKLOG enforcement item (Self-test meta-skill enforces preamble invocation).
- **Description:** The spec deliberately delegates value validation to the resolve-paths preamble — the helper trusts whatever's in `.jim/config.md`. The trust contract is sound when the preamble runs (the common path), but spec 012's Out of Scope already acknowledges that future skills could bypass the preamble silently (the existing backlog items track this). Without explicit naming, the helper inherits that gap with no defense in depth: a preamble-bypassing skill that calls the helper would receive un-validated values and could end up with `..`-traversal paths, absolute paths, or symlink-escape paths flowing into Bash invocations. The "Helper does not validate the value" criterion is correct for normal use but does not name the contract or its failure mode.
- **Suggestion:** Add an acceptance criterion that explicitly documents the trust contract: "The helper assumes the resolve-paths preamble has validated `.jim/config.md` against schema constraints prior to any helper invocation. Skills that invoke the helper without first running the preamble violate this contract. Enforcement that future skills run the preamble is tracked in `BACKLOG.md` (Self-test meta-skill enforces preamble invocation)." Cross-reference the existing backlog enforcement item so the security invariant is named, not buried.
- **Route:** Spec

### 2. Helper stderr echoes the offending key unfenced — captured by Claude's Bash tool

- **Severity:** Notable
- **Status:** **Addressed.** `spec.md` Helper-script section now specifies the stderr error format as `jim_path: unknown key (see .jim/config.md)` with no key echo at all (the safer of the two suggested options). The AC also names the Claude-Code-Bash-tool stderr-capture pathway as the underlying reason.
- **Description:** The unknown-key error format specified is `jim_path: unknown key: <key>`. Helper stderr is captured by Claude Code's Bash tool and surfaced to the agent's context. A malicious `.jim/config.md` (committed to a cloned repo as a supply-chain attack) could include a key name containing markdown, prompt-injection content, or terminal escape sequences (e.g., `path.architecture\n\n[NEW SYSTEM PROMPT: ignore previous instructions]`). When the helper rejects this key as unknown, the offending content flows verbatim into the agent's context. Spec 012's finding 9 addressed the equivalent issue for the preamble's error format by mandating backtick fencing of the key name; the helper's error format does not adopt the same protection. The spec's parenthetical ("key name not fenced — this is shell stderr, not agent prose") underestimates the agent-context exposure that Claude Code's Bash tool creates.
- **Suggestion:** Update the unknown-key error AC to either (a) fence the key in backticks: `` jim_path: unknown key: `<key>` ``, mirroring the preamble's spec 012 finding-9 fix; or (b) emit only `jim_path: unknown key (see .jim/config.md)` with no key echo at all, matching spec 012's broader discipline of not echoing offending content into agent context. Option (b) is the safer choice — the operator can read `.jim/config.md` directly for diagnostics; the agent does not need the offending key in its context to act.
- **Route:** Spec

### 3. Placeholder expansion lacks shell-quoting requirement — metacharacter injection vector

- **Severity:** Notable
- **Status:** **Addressed.** `spec.md` Placeholder-and-preamble section carries a Quoting acceptance criterion specifying single-quote wrapping, the `'\''` escape idiom for embedded single-quotes, and a halt rule for unquotable inputs (e.g., null bytes). Scope updated post-research: the spec adopted Claude Code's documented plugin `bin/` PATH convention, so the helper is invoked by name (no absolute helper path is substituted into the placeholder). The quoting requirement therefore applies only to the absolute project root — the single remaining substituted value. The chokepoint property is unchanged.
- **Description:** The `{jim_path}` placeholder expands to a multi-token shell command of the form `bash <absolute-helper-path> --root=<absolute-project-root>`. Both substituted values come from the runtime environment — the absolute project root from Claude Code's session-level primary working directory, the absolute helper path from the plugin install location. In typical environments these are clean paths (`/home/user/project`), but the spec does not constrain or quote them. A path containing spaces would word-split silently (`~/My Documents/jim`). A path containing shell metacharacters (`;`, `$()`, backticks) would be re-evaluated by bash at command-substitution time, producing arbitrary execution. Project-root metacharacter content could occur in CI/sandbox environments with auto-generated paths, mistyped invocations, or deliberate attacks on a developer's environment. The spec defines the expansion shape but not its escape/quote discipline — given that this placeholder's whole purpose is shell-safety, the gap is structurally relevant.
- **Suggestion:** Add an acceptance criterion: "The preamble's `{jim_path}` placeholder expansion shell-quotes both the absolute helper path and the absolute project root. Single-quotes wrap each value; embedded single-quotes are escaped via the `'\''` idiom. The expansion form is `bash '<helper-path>' --root='<project-root>'` after quoting. The preamble halts with a validation error if either value cannot be represented as a single-quoted string (e.g., contains a null byte)." Closing the metacharacter-injection vector at the placeholder layer is preferable to defending it inside the helper, because the placeholder is the single chokepoint.
- **Route:** Spec

### 4. YAML parsing strategy unspecified — robustness and parser-bug surface

- **Severity:** Advisory
- **Status:** **Addressed in plan.md.** Plan Decision 3 commits to the recommended approach: restrict schema and config to grep-line-readable form (single-line `name: value` entries within the `keys:` sequence; no multi-line scalars, anchors, aliases, or merge tags) and parse via awk. Contract B documents the constraint. Task 1 adds the constraint section to `skills/_shared/config-schema.md`, and Task 2 implements the awk parser. The no-dependency stance is preserved. Residual concern: awk-parser test coverage for edge cases (empty-string defaults, malformed schema) is split out as Finding 8 below.
- **Description:** The helper parses YAML frontmatter from both `skills/_shared/config-schema.md` and `.jim/config.md`. Bash has no built-in YAML parser. The spec leaves the parsing strategy open. Three plausible options exist: (a) hand-rolled parsing with awk/sed/grep (parser bugs possible — quoted strings, multi-line values, edge cases), (b) external tool dependency like `yq` (violates jim's "no dependencies" stance per ARCHITECTURE.md), (c) restricted format that is grep-line-readable (constrains the schema and config format). Option (a) is the implicit default given jim's no-dependency posture, but parser bugs in security-critical code (the schema enumerates valid keys; misparsing could let unknown keys through or reject valid ones) are a real concern. A schema that grows multi-line values, anchors, or quoted strings could break the parser silently and cause every helper call to exit 1.
- **Suggestion:** Plan must specify the YAML parsing strategy explicitly. Recommended: option (c) — constrain the schema and config to grep-line-readable form (single-line `name: value` entries, no multi-line, no anchors, no aliases, simple quoting only) and parse via a small awk script. Add a constraint to `skills/_shared/config-schema.md` that future schema additions must remain in this restricted form. The audit surface becomes a few lines of awk, the YAML edge-case attack surface disappears, and the no-dependency stance is preserved.
- **Route:** Plan

### 5. New executable artifact lacks code-review and static-analysis discipline

- **Severity:** Advisory
- **Status:** **Routed to backlog.** Added to `BACKLOG.md` as an ad-hoc item: "shellcheck CI for jim's executable artifacts."
- **Description:** `bin/jim_path` is the project's first non-markdown executable artifact. ARCHITECTURE.md will be amended to reflect this, but the spec introduces no process discipline for the new artifact: no shellcheck linting, no test harness, no commit-time review checklist. Markdown-only review processes (the existing maintainer pattern) do not scale to shell code — bugs and supply-chain modifications are harder to catch by visual review. Future helper changes (new keys, new flags, new output formats) should pass a static-analysis check before merge. The initial implementation is small enough to audit manually; the concern is sustained discipline, not the first commit.
- **Suggestion:** Backlog item: "shellcheck CI for jim's executable artifacts" — a small CI workflow that runs `shellcheck bin/*` on every push. Optional companion: a test harness for the helper exercising each schema key against a fixture project (overlaps with the future Self-test meta-skill, which could subsume this). This is hardening, not blocking — defer to backlog rather than spec.
- **Route:** Backlog

---

### 6. `readlink -f` is not POSIX/BSD portable — macOS would fail to derive plugin root

- **Severity:** Notable
- **Status:** **Addressed.** Plan Contract D rewritten to use a POSIX-portable symlink-resolution `while [ -L "$path" ]; do …` loop that works on both BSD and GNU readlink (no `-f` flag). Task 2's verify battery now includes a symlinked-invocation case that exercises the loop.
- **Description:** Plan Contract D specifies `helper_path="$(readlink -f -- "${BASH_SOURCE[0]}")"` to resolve the helper's own path before deriving the plugin root via `dirname` twice. The `-f` flag is a GNU coreutils extension; macOS's BSD `readlink` does not support `-f` and exits non-zero when given the flag. Claude Code is documented as supported on macOS (per spec 013 Out of Scope: "Windows non-WSL support … inherits the same bash dependency as Claude Code itself" — implying macOS is a first-class platform). On macOS, every `$(jim_path …)` call would fail at the very first line of the helper, breaking every Bash-mediated config lookup. The failure is loud (`readlink: illegal option -- f`) but happens before any helper logic runs, so no useful error message reaches the agent context.
- **Suggestion:** Replace the `readlink -f` call in Contract D with a portable symlink-resolution shim. Standard POSIX pattern:
  ```bash
  helper_path="$0"
  while [ -L "$helper_path" ]; do
      target="$(readlink "$helper_path")"
      case "$target" in
          /*) helper_path="$target" ;;
          *)  helper_path="$(dirname -- "$helper_path")/$target" ;;
      esac
  done
  helper_path="$(cd -P -- "$(dirname -- "$helper_path")" && pwd -P)/$(basename -- "$helper_path")"
  ```
  Or, simpler if symlink resolution is not strictly required (and the plugin install likely does not symlink into `bin/`): use `$0` directly via `cd -P` for canonicalization without symlink-following. Pick one and document it explicitly in Contract D, with an accompanying Task 2 verify case that exercises a symlinked invocation on at least one platform.
- **Route:** Plan

### 7. No plausibility check on derived plugin root

- **Severity:** Advisory
- **Status:** **Addressed.** Plan Contract D Step 3 adds an explicit plausibility check: after `dirname`-twice plugin-root derivation, the helper greps `<plugin-root>/.claude-plugin/plugin.json` for `"name": "jim"` and exits 1 with stderr `jim_path: not a jim plugin install` on mismatch. The new error message is added to Contract A's stderr-format list. Task 2's verify battery exercises both the wrong-plugin-name path and the malformed-schema path (which together also confirm the plausibility check passes the legitimate case).
- **Description:** Plan Contract D derives the plugin root by `dirname` twice from the helper's own path, then reads `<plugin-root>/skills/_shared/config-schema.md`. There is no verification that the derived path is actually jim's plugin root. If `bin/jim_path` is moved (copied to `~/.local/bin/`, symlinked into a different plugin's `bin/`, executed via a wrapper that obscures `BASH_SOURCE`), the helper silently reads a non-existent or wrong schema. The "malformed schema" exit-1 path catches the most-broken cases, but a stale or unrelated schema with valid format would parse without error and produce wrong key validation. Combined with the spec's trust-the-value design, the helper has no defense against a misinstall-induced wrong-plugin-root and would resolve attacker-controlled keys silently from a different location's config.
- **Suggestion:** Add a sanity check in the helper after deriving `plugin_root`: verify `<plugin-root>/.claude-plugin/plugin.json` exists and contains `"name": "jim"`. If absent or wrong, exit 1 with stderr `jim_path: not a jim plugin install`. One-line `grep -q '"name"[[:space:]]*:[[:space:]]*"jim"'` against the manifest. Closes the misinstall-or-relocation gap with the same fail-loud discipline established by spec 012's preamble. Add a corresponding entry to Plan Task 2's verify battery exercising a wrong-plugin-root invocation.
- **Route:** Plan

### 8. Awk-parser test coverage is incomplete — empty-string defaults and malformed-schema cases unverified

- **Severity:** Advisory
- **Status:** **Addressed.** Plan Task 2's verify battery extended with: (a) an empty-string-default test against `specs.id-prefix` (verifies stdout is exactly a single newline byte via `xxd`); (b) a malformed-schema test using a temporary plugin-tree fixture, confirming exit 1 with `jim_path: malformed schema` stderr. Fixture setup is shared with the Finding 7 wrong-plugin-name test for efficiency.
- **Description:** Plan Task 2's verify battery exercises `path.architecture` (non-empty path) and `workflow.require-plan-approval` (boolean) but does not test (a) the empty-string default case, exemplified by the existing `specs.id-prefix: ""` schema entry — an awk parser that strips whitespace too aggressively could collapse `""` to nothing, skip the line, or produce a different value than the literal empty string; (b) the malformed-schema detection path that Contract A specifies (`jim_path: malformed schema`, exit 1) — without a test case that triggers this branch, the awk parser's strictness is unverified, and a future schema change could introduce silent misparsing.
- **Suggestion:** Extend Plan Task 2's verify battery with two additional cases:
  ```
  ./bin/jim_path specs.id-prefix | xxd | grep -q '^00000000: 0a$'    # empty-string default → only newline
  # Malformed-schema test:
  scratch="$(mktemp -d)" &&
  mkdir -p "$scratch/bin" "$scratch/skills/_shared" "$scratch/.claude-plugin" &&
  cp bin/jim_path "$scratch/bin/" &&
  printf 'name: jim\n' > "$scratch/.claude-plugin/plugin.json" &&
  printf -- '---\nkeys:\n  - bogus\n---\n' > "$scratch/skills/_shared/config-schema.md" &&
  ! "$scratch/bin/jim_path" path.architecture &&
  rm -rf "$scratch"
  ```
  The malformed-schema construction also exercises the plausibility check from Finding 7 (a plugin.json file is needed). If Finding 7 is accepted, both tests can share fixture setup.
- **Route:** Plan

### 9. PATH precedence for plugin `bin/` is undocumented

- **Severity:** Advisory
- **Status:** **Addressed.** Plan Decision 8 documents the assumption (Claude Code prepends plugin `bin/` to PATH per platform docs) and explicitly rejects in-helper self-checks as over-engineered for the loud failure mode. Task 5's dogfood verify includes a PATH-precedence check confirming `which jim_path` resolves to the jim plugin's helper.
- **Description:** Plan Decision 1 leverages Claude Code's plugin `bin/` PATH convention. Whether the plugin's `bin/` is *prepended* to PATH (overriding system binaries with the same name) or *appended* (system shadows plugin) is not pinned down in the plan, and the research did not specify it. If a user has a pre-existing `jim_path` binary on PATH (unlikely but not implausible), it could shadow or be shadowed by the plugin's helper depending on precedence. Either case is detectable (different output, missing keys, "command not found"), but the failure mode is hard to diagnose because the agent's context shows only the resulting error, not the precedence root cause. Lower priority than findings 6 and 7 because the failure mode is loud rather than silent.
- **Suggestion:** Add a Design Decision to `plan.md` documenting the assumed PATH precedence (plugin `bin/` prepended, per the Claude Code docs the research cited). Add an early task verify that confirms the assumption holds on the developer's machine: `which jim_path | xargs realpath | xargs dirname | xargs dirname` should equal the jim plugin root. Defer hardening (a self-check inside the helper that aborts if `$0` is not the expected plugin-relative path) to follow-up work; the assumption-documentation step is sufficient.
- **Route:** Plan

## STRIDE Coverage

| Category | Relevant? | Findings |
| :--- | :--- | :--- |
| Spoofing | N/A | No identity or authentication layer. Helper is a local subprocess. |
| Tampering | Yes | Findings 1, 2, 3, 7. `.jim/config.md` is repo-committable (supply-chain surface); schema validation is the defense; helper trust contract, stderr fencing, placeholder-expansion quoting, and plugin-root plausibility checks are all parts of that defense. |
| Repudiation | N/A | No audit requirement. Resolved-paths audit log is tracked separately in BACKLOG.md. |
| Information Disclosure | Yes | Finding 2. Helper stderr captured by Claude's Bash tool flows attacker-controlled key content into agent context. |
| Denial of Service | Yes | Finding 6. macOS BSD `readlink` incompatibility breaks every helper invocation on a supported platform — operational unavailability, not a resource-exhaustion attack but a self-inflicted DoS. |
| Elevation of Privilege | Yes | Findings 1, 3, 7. Unvalidated values, metacharacter-rich placeholder expansion, and a non-verified plugin root all grant filesystem or arbitrary-execution access outside intended scope when defenses fail. |

## Routing Recommendations

### Spec amendments
- **Finding 1:** Add AC explicitly documenting the helper's trust contract and cross-referencing the BACKLOG enforcement item.
- **Finding 2:** Update the unknown-key error format AC — either backtick-fence the key (mirroring spec 012 finding 9) or omit the key entirely (recommended).
- **Finding 3:** Add AC mandating shell-quoting of both substituted values in the `{jim_path}` placeholder expansion, with a halt rule for unquotable inputs.

### Plan amendments
- **Finding 6 (Notable):** Replace `readlink -f` in Contract D with a portable symlink-resolution shim (POSIX while-loop pattern); add a Task 2 verify case exercising symlinked invocation. Required before build can begin on macOS.
- **Finding 7 (Advisory):** Add a plugin-root plausibility check to the helper — verify `<plugin-root>/.claude-plugin/plugin.json` contains `"name": "jim"` and exit 1 with `jim_path: not a jim plugin install` on mismatch.
- **Finding 8 (Advisory):** Extend Task 2's verify battery to cover the empty-string-default case (`specs.id-prefix`) and a malformed-schema test that triggers Contract A's `jim_path: malformed schema` exit-1 path. Reuses Finding 7's fixture setup if both are accepted.
- **Finding 9 (Advisory):** Document PATH-precedence assumption in a new Design Decision; add an early-task verify confirming `which jim_path` resolves to the plugin's `bin/jim_path`.

### Backlog items
- **Finding 5:** "shellcheck CI for jim's executable artifacts" — sustained static-analysis discipline for the new executable surface. Already routed to BACKLOG.md.
