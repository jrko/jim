# Security Review Definition of Done

Self-check reference for the security agent and the `/jim:sec` skill. Every security review must pass all applicable items before presentation.

## Checklist

### Findings Quality

1. **Complete findings:** Every finding includes all four fields: severity (Critical/Notable/Advisory), description, suggestion, and route (Spec/Plan/Backlog).

2. **Actionable suggestions:** Every suggestion is concrete and specific — not vague ("consider security") or aspirational ("should be more secure"). The recipient (PM, architect, or backlog) can act on it without further clarification.

3. **Correct severity:** Critical = design flaw that will create a vulnerability if built as-is. Notable = gap that should be addressed before build. Advisory = hardening opportunity, good candidate for backlog.

4. **Correct routing:** Critical and Notable findings route to Spec or Plan. Advisory findings route to Backlog. Routing matches the nature of the finding — requirements gaps route to Spec, design flaws route to Plan.

5. **No duplicates:** Findings do not duplicate security considerations already documented in `ARCHITECTURE.md`. If a finding reinforces an existing constraint, reference it rather than restating it.

### Analysis Coverage

6. **Freeform review completed:** Expert review was performed before the STRIDE sweep. Context-specific, non-obvious issues were considered first.

7. **STRIDE sweep completed:** All six STRIDE categories are listed in the coverage table. Each is marked as relevant, not relevant, or N/A with justification. Irrelevant categories are explicitly skipped — not silently omitted.

8. **Architecture-grounded:** If `ARCHITECTURE.md` exists, it was read and findings are grounded in the project's existing trust boundaries, data flows, and security patterns.

9. **Lens-appropriate:** Spec-phase analysis focuses on requirements gaps (missing controls, unaddressed boundaries, data classification). Plan-phase analysis focuses on design flaws (flawed mitigations, privilege issues, crypto choices). Both lenses applied when both artifacts exist.

### Output Format

10. **Linked:** Frontmatter `spec:` field contains the relative path to the source spec.md, or `"ad-hoc"` for non-spec reviews.

11. **Status set:** Frontmatter `status:` is one of: `Active` (no upstream concerns), `Needs Spec Review` (findings require spec changes), `Needs Plan Review` (findings require plan changes).

12. **Routing section present:** Routing recommendations section lists findings organized by destination (Spec/Plan/Backlog). Empty destination sections are removed.

### Mode-Specific Checks

13. **Spec-scoped mode:** Output is written to `security.md` in the spec directory as a sibling artifact. Routing offers spec, plan, and backlog destinations.

14. **Ad-hoc mode:** Output is delivered in conversation only — no file written. Routing offers backlog destination only.

15. **Differential update:** When `security.md` already exists, the existing file is read first, changes are summarized, and Edit is used (not Write) to update.
