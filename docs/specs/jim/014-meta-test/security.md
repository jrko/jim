---
spec: "spec.md"
status: Active
date: "2026-04-29"
---

# Security Review: Meta-test skill — static audit of config-adherence invariants

## Summary

Differential review covering spec and plan. All seven spec-phase findings (1–7) are **addressed**: findings 1–6 amend `spec.md` AC and acceptance language; finding 7 was routed to `BACKLOG.md`. Three new plan-phase findings (8–10) are surfaced from `plan.md`: one notable (schema-read failure semantics undefined), two advisory (schema-trust boundary not documented; audit-surface symlink policy not specified). No critical findings. STRIDE relevant only for Tampering — consistent with the spec-phase conclusion.

**Artifacts reviewed:**
- `docs/specs/jim/014-meta-test/spec.md` (post-amendment, status: approved)
- `docs/specs/jim/014-meta-test/plan.md` (new — 13-task feature plan with 5 interface contracts and 8 design decisions)
- `docs/specs/jim/014-meta-test/research.md` (new — codebase grounding)
- `ARCHITECTURE.md` (Security Considerations, Plugin Executables, Shared Primitives)
- `VISION.md` (strategic context)
- `skills/_shared/config-schema.md`, `skills/_shared/resolve-paths.md` (referenced by spec and plan)
- `docs/specs/jim/012-config-adherence/security.md` (prior security findings the meta-test inherits enforcement of)
- Prior `security.md` (this file, superseded by this review)

## Findings

### 1. "Schema value constraints" check is too narrow

- **Severity:** Notable
- **Description:** The third invariant check is scoped to "every `path.*` key declared in `skills/_shared/config-schema.md` carries the documented value-type rules (relative path, no `..` segments after normalization, no leading `/`, resolves within project root)." But `config-schema.md`'s Validation Rules section defines four other rules: `positive-integer` (must be ≥ 1), `boolean` (literal `true`/`false`; case variants and YAML 1.1 forms rejected), `string` (unconstrained), and the unknown-key hard-error rule. A future contributor weakening the boolean rule (e.g., re-allowing `yes`/`no`) or removing the unknown-key rule would not be caught. The meta-test should defend the entire schema's value-rule surface, not just `path.*` constraints.
- **Suggestion:** Broaden the AC bullet to "every value-type rule documented in `config-schema.md`'s Validation Rules section is preserved across schema edits — `path.*` constraints, `positive-integer`, `boolean` (literal `true`/`false` only, case variants and YAML 1.1 forms rejected), `string`, and the unknown-key hard-error rule. The check verifies presence of each rule, not specific wording."
- **Route:** Spec
- **Status:** **Addressed.** Spec AC bullet "Schema value rules" lists all five rules. Plan Decision 2 (schema-derived corpus) and task 6 specify deriving the rule list from the schema's `## Validation Rules` section. Plan task 6 verify greps for each of the five rule names.

### 2. "Preamble invocation" check phrasing is ambiguous

- **Severity:** Notable
- **Description:** The first invariant check reads "every `skills/*/SKILL.md` body invokes `skills/_shared/resolve-paths.md` at step 1 via the canonical reference phrasing." "Canonical reference phrasing" is undefined. If interpreted as exact-phrase match, the check becomes brittle (minor rephrasing breaks it, including author-style edits that don't change meaning). If interpreted as any prose mention, the check becomes shallow (an attacker can write a reference Claude sees but a human reviewer might miss, e.g., a comment-style mention without runtime weight). The robust framing is structural: the file path `skills/_shared/resolve-paths.md` must appear inside step 1 of the SKILL.md, not anywhere else.
- **Suggestion:** Restate the AC bullet as "every `skills/*/SKILL.md` step 1 contains a literal reference to the file path `skills/_shared/resolve-paths.md`." File-path references are structural — rephrasing the surrounding prose doesn't break the check, but a missing or relocated reference does. Spec body should also include a positive and a negative example to anchor Claude's judgment.
- **Route:** Spec
- **Status:** **Addressed.** Spec AC bullet "Preamble invocation" now reads structurally: "every `skills/*/SKILL.md`'s step 1 contains a literal reference to the file path `skills/_shared/resolve-paths.md`." Plan task 5 implements; Contract D documents the structural rule; anchor examples (positive + negative) are required by the AC and produced in plan task 5.

### 3. `$(jim_path …)` substitution check should require the `{jim_path}` placeholder form

- **Severity:** Notable
- **Description:** The fourth invariant check reads "every Bash invocation in the audit surface that references a configurable path uses `$(jim_path KEY)` substitution rather than a literal default filename or hardcoded path." Spec 013 introduced the `{jim_path}` derived placeholder which expands to `jim_path --root='<absolute-project-root>'` for cd-safety; skill bodies are required to compose `$({jim_path} <key>)`, not raw `$(jim_path <key>)`. A skill author writing the literal helper name without the placeholder would silently bypass the cd-safe `--root=` flag injection, defeating spec 013's runtime guarantee. The current AC wording would not catch that regression because it accepts any `$(jim_path KEY)` shape.
- **Suggestion:** Tighten the AC bullet to "every Bash invocation in the audit surface that references a configurable path uses the placeholder form `$({jim_path} <key>)` — not a literal `$(jim_path <key>)` invocation, not a hardcoded path, and not an alternative resolution mechanism such as inline `awk` parsing of `.jim/config.md`. KEY must be a config key declared in `config-schema.md`'s `keys:` frontmatter list." Optionally clarify that "configurable path" means a path corresponding to a schema-declared key, not arbitrary hardcoded paths in unrelated examples.
- **Route:** Spec
- **Status:** **Addressed.** Spec AC bullet now requires the placeholder form `$({jim_path} <key>)` and explicitly flags raw `$(jim_path <key>)`, hardcoded paths, and alternative mechanisms (inline awk). Plan task 8 implements all three negative cases in the skill body; Contract A frontmatter `description` re-states the placeholder requirement. "Configurable path" scope is plan-task-8 explicit.

### 4. Self-audit by inclusion is not explicit

- **Severity:** Advisory
- **Description:** The meta-test audits `skills/*/SKILL.md`, which by glob includes `skills/meta-test/SKILL.md`. So self-audit is implicit — but the spec does not say so. A future contributor adding "exclude self to avoid bootstrap weirdness" to the audit surface would silently disable the self-check, eroding the meta-test's integrity guarantee. Making the property explicit prevents that drift.
- **Suggestion:** Add an AC bullet: "The meta-test audits its own `skills/meta-test/SKILL.md` along with all other audited skills. Self-exclusion is prohibited; the meta-test's own preamble invocation, literal-default-filename position, and any Bash invocations are checked under the same rules as every other skill." Optionally add a sentence to the Out of Scope section noting that self-modification (writing back to its own SKILL.md) is out of scope, distinct from self-audit.
- **Route:** Spec
- **Status:** **Addressed.** Spec AC bullet "Self-audit by inclusion" added; spec Out of Scope adds "Self-modification" entry distinct from self-audit. Plan Decision 6 (no self-exclusion mechanism) implements; plan task 4 verify includes `audits itself`/`self-audit` grep; plan task 12 self-lint sanity pass exercises the meta-test against its own body.

### 5. `/jim:config` exemption scope should be narrowed to scaffolding instructions

- **Severity:** Advisory
- **Description:** The AC currently exempts `skills/config/SKILL.md` from the literal-default-filename check "for cases where `/jim:config` is scaffolding default values into a new `.jim/config.md`." Read literally, that's the right intent. Read loosely, an implementer might extend the exemption to all literals in `skills/config/SKILL.md` — including procedural Read/Write steps that happen to mention default filenames in non-scaffolding contexts. That widens the gap.
- **Suggestion:** Tighten the exemption wording to "Literals appearing inside the scaffolding instructions for new `.jim/config.md` files are exempt — defined as the section of `skills/config/SKILL.md` that emits default key/value pairs into the user's project config. Procedural Read/Write/Glob steps elsewhere in `skills/config/SKILL.md` are subject to the same literal-default-filename check as every other audited skill." Plan can pin the exemption boundary to specific section anchors when the skill is built.
- **Route:** Spec
- **Status:** **Addressed.** Spec AC narrows the exemption to "scaffolding sections of `skills/config/SKILL.md`." Plan Decision 4 + Contract E pin the boundary to `skills/config/SKILL.md` steps 3 and 4; steps 1, 2, 5 are subject to normal Check 3 rules. Plan task 7 implements the exemption with explicit step numbers in the skill body.

### 6. Judgment-based check needs a documented bias direction

- **Severity:** Advisory
- **Description:** The third invariant check ("tool-argument literal defaults") explicitly relies on Claude applying judgment to distinguish prose mentions from tool-argument positions. Judgment-based checks have variance — two runs over the same source may disagree on edge cases. Without a documented bias direction, the meta-test's pass/fail signal is non-deterministic in the gray zone, and a contributor cannot tell whether a green run reflects clean source or favorable interpretation. The reasonable bias is false-positive (flag when ambiguous) rather than false-negative (pass when ambiguous), because the cost of a false flag is one prose tweak while the cost of a missed leak is a config-adherence regression.
- **Suggestion:** Add to the AC: "When the literal-default-filename check is ambiguous (the literal could plausibly be either prose or a tool argument), flag it. Bias is toward false-positive — a contributor can resolve a false flag by rephrasing or by using a non-default illustrative value, but a missed leak goes silent. Anchor examples in the skill body should illustrate both clearly-prose and clearly-tool-argument cases to calibrate the judgment."
- **Route:** Spec
- **Status:** **Addressed.** Spec AC bullet "Tool-argument literal defaults" includes the explicit "Bias is toward false-positive" clause. Plan task 7 implements the bias rule in the skill body and requires positive (clearly-prose) and negative (clearly-tool-argument) anchor examples per Decision 5.

### 7. Schema restricted-YAML format constraint not audited

- **Severity:** Advisory
- **Description:** `config-schema.md`'s Schema Format Constraint section requires the schema and any `.jim/config.md` to conform to a restricted YAML subset (single-line scalars, no anchors/aliases/merges, three-field key entries) so that `bin/jim_path`'s hand-rolled awk parser remains correct. A future contributor adding a schema entry that violates this constraint would not break the meta-test's checks but would silently break the helper at runtime. The helper itself exits with `malformed schema` in that case — the runtime defense is in place. But a meta-test that statically verified the constraint would catch the regression at PR time rather than first invocation.
- **Suggestion:** Defer to a follow-up spec or backlog item. The runtime defense in `bin/jim_path` already handles this case; static enforcement is a hardening step rather than a requirements gap. Add to backlog as "Meta-test: verify schema restricted-YAML format constraint statically" with `docs/specs/jim/014-meta-test/security.md` finding 7 as the source.
- **Route:** Backlog
- **Status:** **Addressed.** Routed to `BACKLOG.md` ad-hoc section as "Meta-test verifies schema restricted-YAML format constraint" with this finding cited as source. Plan Out of Scope acknowledges the deferral.

---

### 8. Schema-read failure semantics undefined in plan

- **Severity:** Notable
- **Description:** Plan Decision 2 + task 3 require the meta-test to Read `skills/_shared/config-schema.md` at runtime to derive (a) the literal-default search corpus and (b) the type-rule presence checklist. The plan does not specify what the meta-test does if the schema is missing, unreadable, or malformed (e.g., the `keys:` frontmatter has been corrupted, or someone deleted the `## Validation Rules` section). Three possible behaviors — halt with an error, silently fall back to a hardcoded list, or proceed with an empty corpus — produce very different security postures. Silent fallback weakens the audit (Decision 2 deliberately rejected hardcoded lists for this reason); proceeding with an empty corpus would silently pass every Check 3 (no defaults to search for) and Check 2 (no rules to verify); only fail-loud preserves the meta-test's value as a defense.

  This is the same hazard `bin/jim_path` mitigates with its `malformed schema` exit-1 path. The meta-test's runtime is Claude reading the schema text rather than awk parsing it, so the failure modes differ — but the principle is the same: a broken schema must produce a loud, blocking signal, not a silent ✓.
- **Suggestion:** Add a plan task (or extend task 3) specifying the meta-test's failure semantics when reading the schema:
  1. **Schema file missing:** halt with an in-conversation error like `✗ /jim:meta-test — cannot read skills/_shared/config-schema.md`. Do not proceed to checks.
  2. **`keys:` frontmatter missing or unparseable as a mapping sequence:** halt with `✗ /jim:meta-test — schema keys: section is missing or malformed`. Do not proceed.
  3. **`## Validation Rules` section missing:** halt with `✗ /jim:meta-test — schema Validation Rules section is missing`. Do not proceed.
  4. **Schema readable but `keys:` empty (no path keys):** Check 3 reports `0/0` legitimately; do not halt.

  Document this in the skill body (step 2) and add a verify in the relevant plan task asserting the skill body contains halt-on-schema-failure language.
- **Route:** Plan
- **Status:** **Addressed.** Plan task 3 now specifies the four failure semantics (missing/unreadable schema, missing/malformed `keys:`, missing `## Validation Rules`, empty `keys:`) with the exact halt messages. Task 3 verify command extended to grep for `halt`/`fail-loud`/`cannot read` and the literal "schema Validation Rules section is missing" string. Skill body explicitly prohibits silent fallback or hardcoded-list compensation.

### 9. Schema-trust boundary not documented in plan or skill prose

- **Severity:** Advisory
- **Description:** The meta-test's correctness depends entirely on `skills/_shared/config-schema.md` being well-formed and complete. If a malicious commit modifies the schema to remove a key (e.g., deletes the `path.architecture` entry), Check 3 will no longer search for `ARCHITECTURE.md` as a literal default, and a coordinated leak in a skill body would pass undetected. This is a defense-in-depth concern: the meta-test treats the schema as authoritative input. The architecture document already flags `_shared/` as security-relevant, so this trust relationship exists by design — but the plan and skill prose should acknowledge it explicitly so future contributors understand that schema modifications are themselves a security event, not just a configuration change.
- **Suggestion:** Add a one-paragraph note to plan Decision 2 (or to the skill body's step 2) stating: "The meta-test trusts `skills/_shared/config-schema.md` as the authoritative source of configurable keys and validation rules. Tampering with the schema (removing a key, weakening a rule) effectively disables the corresponding part of the audit. Schema modifications carry the same review weight as modifications to `bin/jim_path` or `resolve-paths.md` — they are part of the security-relevant trio per `ARCHITECTURE.md`." This is documentation, not a runtime change.
- **Route:** Plan
- **Status:** **Addressed.** Plan Decision 2 now carries a "Trust boundary" paragraph documenting the schema-as-authoritative-input relationship, the failure modes that disable the audit, and the security-relevant-trio framing. The paragraph cites task 3's fail-loud semantics as the defense-in-depth mechanism.

### 10. Audit-surface symlink policy unspecified

- **Severity:** Advisory
- **Description:** Plan task 4 instructs the agent to Glob `skills/*/SKILL.md` and `agents/*.md` for the in-scope file set. If a malicious or careless commit introduces a symlink under `skills/` or `agents/` (e.g., `skills/evil/SKILL.md → ../../some/external/file`), the Glob would match it and Read would dereference it. Three failure modes follow: (a) the audit reads content outside the intended audit surface; (b) symlink loops or paths to large files could exhaust Claude's tool budget mid-audit; (c) a symlink targeting a known-clean SKILL.md elsewhere could mask a real violation by aliasing to it. None of these are likely in jim's tightly-controlled repo, but the policy is undefined. The same hazard surfaces in `config-schema.md`'s path validation (which uses realpath semantics + project-root containment), and the meta-test should follow the same posture.
- **Suggestion:** Add a one-line constraint to plan Contract D and the skill body: "Symlinks under `skills/` and `agents/` are not part of jim's repo discipline; if the audit encounters one (a Glob match whose realpath lies outside the project root or whose target is not a regular file), halt with an error. Repo-level review is the primary defense; the meta-test does not attempt to dereference symlinks." Implementation can be deferred — even a TODO note in the skill body counts as documenting the policy. Alternative: route the implementation to BACKLOG.md and document the assumption in the plan ("audit assumes no symlinks in the audit surface; enforcement deferred").
- **Route:** Backlog
- **Status:** **Addressed.** Routed to `BACKLOG.md` as "Meta-test: audit-surface symlink policy" with this finding cited as source. Plan does not block on this — repo-review discipline is the current defense.

## STRIDE Coverage

| Category | Relevant? | Findings |
| :--- | :--- | :--- |
| Spoofing | N/A | No identity layer; the skill is read-only and runs under the developer's Claude Code session permissions. |
| Tampering | Yes | Findings 1, 2, 3 (spec phase) sharpen the audit's resistance to drift in the security-relevant trio's invariants. Findings 8, 9, 10 (plan phase) extend the same posture to the schema-as-input trust boundary, schema-read failure semantics, and the audit-surface symlink policy. |
| Repudiation | N/A | No audit-log requirement. Report is ephemeral by design (in-conversation only, no artifact). |
| Information Disclosure | N/A | The fail-case report quotes file contents; jim is open-source so excerpting source text is not a disclosure concern. Forks with proprietary skill prose would inherit a marginal concern, but that is out of scope for the upstream spec. |
| Denial of Service | Marginal | Symlink loops in the audit surface (finding 10) could in principle exhaust Claude's tool budget mid-audit. Mitigated in practice by repo-review discipline and Claude Code's tool-call ceilings; no separate runtime guard in the plan. |
| Elevation of Privilege | N/A | Read-only skill; no writes, no execution, no privilege boundary crossed beyond what the developer's session already has. |

## Routing Recommendations

### Spec amendments (closed)

All six spec-phase findings (1–6) were routed to spec and addressed in the post-review spec amendment. Status: closed.

### Plan amendments (closed)

Plan-phase findings 8 and 9 routed to plan and addressed in the post-review plan amendment. Status: closed.

### Backlog items

- Finding 7: meta-test should statically verify the schema's restricted-YAML format constraint. Source: `docs/specs/jim/014-meta-test/security.md` finding 7. *(Added to `BACKLOG.md`.)*
- Finding 10: audit-surface symlink policy — define behavior when the Glob matches a symlink under `skills/` or `agents/`. Source: `docs/specs/jim/014-meta-test/security.md` finding 10. *(Added to `BACKLOG.md`.)*
