---
spec: "spec.md"
status: Active
date: "2026-04-23"
---

# Security Review: Config adherence — placeholder rewrites, shared resolve-paths preamble, schema validation

## Summary

Differential review covering spec and plan. All eleven findings across both review passes are addressed — spec-phase findings 1–5 are concrete acceptance criteria in `spec.md`; plan-phase findings 6–11 are concrete contract amendments and task updates in `plan.md`. No open security items remain.

**Artifacts reviewed:**
- `docs/specs/jim/012-config-adherence/spec.md` (updated since prior review — AC amendments from findings 1–5)
- `docs/specs/jim/012-config-adherence/plan.md` (new — 29-task refactor plan with 5 interface contracts)
- Prior `security.md` (this file, superseded by this review)
- `ARCHITECTURE.md`, `VISION.md`

## Findings

### 1. Path value validation — unconstrained `path.*` values

- **Severity:** Notable
- **Status:** **Addressed.** Plan Contract A defines a `Validation Rules` section requiring `path.*` values to be relative, free of `..` segments after normalization, without a leading `/`, and to resolve inside the project root. Hard-error path is specified in Contract C. Tasks 1 writes the rules; Task 27.2 and 27.3 exercise the `..` and absolute-path negative cases in dogfood.

### 2. Prompt injection via resolved-paths table

- **Severity:** Advisory
- **Status:** **Addressed (and superseded).** The resolved-paths table was removed entirely during post-build cleanup — the placeholder syntax itself is the forcing function and the table emission added noise without proportional value. With no table emission, there is no value-echoing surface to inject into. Finding's underlying risk is moot. Plan Design Decision 5 captures the rationale.

### 3. Error messages echoing raw values

- **Severity:** Advisory
- **Status:** **Addressed.** Plan Contract C mandates the error format name the key and a human-readable reason, with explicit prohibition on echoing the raw offending value. Tasks 27.1–27.3 verify the format.

### 4. `skills/_shared/*` files are security-relevant — label them

- **Severity:** Advisory
- **Status:** **Addressed.** Plan Task 29 extends the post-build `ARCHITECTURE.md` refresh to name `skills/_shared/config-schema.md` and `skills/_shared/resolve-paths.md` as security-relevant with their invariants.

### 5. Future-skill preamble-bypass risk

- **Severity:** Advisory
- **Status:** **Addressed (deferred enforcement).** Spec Out of Scope now explicitly acknowledges the snapshot-only audit; backlog carries the enforcement items (two ad-hoc items in `BACKLOG.md`) for the self-test meta-skill.

---

### 6. Agent tool calls may use unresolved `{path.*}` placeholders as literal paths

- **Severity:** Notable
- **Status:** **Addressed.** Plan Design Decision 7 and Contract E now specify that placeholders are for reasoning and prose only — agents never pass a placeholder string to a `Write`, `Edit`, `Read`, or `Glob` tool call. Two resolution paths are permitted: skill-mediated (the preamble resolves the placeholder internally and substitutes at point of use) or inline-resolved (read `.jim/config.md` and resolve before constructing the tool argument). Tasks 19–24 verify the rule text appears in each agent file.

### 7. Path normalization semantics — symlink handling unspecified

- **Severity:** Notable
- **Status:** **Addressed.** Contract A Validation Rules now specify realpath semantics: "Path normalization follows symlinks. The final resolved target — after following any symlinks in the lexical path — must lie within the project root. Symlinks whose targets lie outside the project root are rejected with the `value must resolve within project root` reason, regardless of whether the lexical path contains `..`." Task 27.4 exercises a committed-symlink escape in dogfood.

### 8. YAML boolean parsing ambiguity for `workflow.*` keys

- **Severity:** Advisory
- **Status:** **Addressed.** Contract A boolean rule tightened to "must be the literal YAML `true` or `false`. Case variants and legacy YAML 1.1 forms (`yes`, `no`, `on`, `off`) are rejected with a `must be literal true or false` reason."

### 9. Validation error echoes the offending key unfenced

- **Severity:** Advisory
- **Status:** **Addressed.** Contract C updated to emit `✗ Key: \`{offending key name}\`` — keys now backtick-fenced to match the value-fencing discipline. Task 27 verify updated to confirm the key appears inside inline backticks.

### 10. `.jim/skills/_shared/` overlay boundary undocumented for users

- **Severity:** Advisory
- **Status:** **Addressed.** Contract A now includes an "Overlay Boundary" section stating: "`skills/_shared/` is part of the jim plugin contract and is not overlayable via `.jim/skills/_shared/`. A user who creates `.jim/skills/_shared/config-schema.md` expecting to add or loosen validation rules will find that file silently ignored."

### 11. Agent whole-file sweep not explicitly required

- **Severity:** Advisory
- **Status:** **Addressed.** Contract E rewritten as "Agent whole-file rewrite template" explicitly covering the Context section, tool-call semantics, and any other prose. Tasks 19–24 rescoped from "rewrite Context section" to "sweep the whole file." Verify commands updated to assert both `{path.*}` presence and the tool-call rule text.

## STRIDE Coverage

| Category | Relevant? | Findings |
| :--- | :--- | :--- |
| Spoofing | N/A | No identity or authentication layer in scope. |
| Tampering | Yes | Findings 1, 7. `.jim/config.md` is user/repo-controlled; schema is the defense; symlink semantics must be specified for the defense to be reliable. |
| Repudiation | N/A | Local developer tooling; no audit requirement. |
| Information Disclosure | Yes | Findings 2, 3, 9. Config values and keys flow into agent context via table and error messages; fencing required consistently. |
| Denial of Service | N/A | Hard-error is intentional fail-loud; expected behavior, not a gap. |
| Elevation of Privilege | Yes | Findings 1, 6, 7. Unbounded path values, agent direct-write to unresolved placeholders, and symlink traversal all grant filesystem access outside intended scope. |

## Routing Recommendations

All eleven findings are addressed. No open routing items.

### Backlog items
- None new in this review. Existing backlog items (two ad-hoc items derived from prior Finding 5) remain.
