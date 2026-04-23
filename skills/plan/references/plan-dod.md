# Plan Definition of Done

Self-check reference for the architect agent and the `/jim:plan` skill. Every plan.md must pass all applicable items before presentation.

## Checklist

### Frontmatter

1. **Linked:** Frontmatter `spec:` field contains the relative path to the source spec.md.

2. **Typed:** Frontmatter `type:` is one of `feature`, `bug`, or `refactor` — carried from the spec.

3. **Drafted:** Frontmatter `status:` is `draft`. Never auto-approve.

### Structure

4. **Overview Present:** Overview section is 1-2 sentences. Summarizes approach and key design choice — not a restatement of the spec.

5. **Design Decisions Documented:** Every non-obvious design choice has a Chosen/Why/Rejected block. Absence of design decisions is a red flag — if every choice was obvious, say so explicitly.

6. **Constitution Check Present:** Either lists `ARCHITECTURE.md` constraints honored by this plan, or notes that `ARCHITECTURE.md` is absent. Never silently skip this section.

7. **File Manifest Complete:** Lists every file that will be created or modified. No surprise files in the task breakdown that aren't in the manifest.

8. **Interface Contracts Before Tasks:** Contracts (types, interfaces, API shapes, schema) are defined in their own section before the task breakdown. Tasks reference the contracts — not the other way around.

9. **Mermaid Diagram Present:** At least one Mermaid diagram for non-trivial data flows or state transitions. Trivial plans (single-file edits) may skip with a note.

### Tasks

10. **Atomic Tasks:** Each task changes one thing and verifies one thing. "Create X, update Y, and test Z" in a single task is a failure — split it.

11. **Verify Commands:** Every task has a `**Verify:**` entry with a shell-executable command. "Check manually" or "review the output" are not valid verifies.

12. **Dependency Order:** Tasks are ordered by dependency. If ordering is non-obvious, notes like "Depends on task N" are present.

13. **Type-Specific Structure:**
    - **Feature:** Standard ordered breakdown.
    - **Bug:** Reproduce → Fix → Regression structure. The regression task verifies the fix doesn't re-break.
    - **Refactor:** Structural changes first. Each task ends with an "existing tests pass" verification.

### Coverage

14. **Requirements Covered:** Every acceptance criterion from the spec appears in the Requirements Coverage Summary table with at least one task reference. An AC with no task is either explicitly Out of Scope or a planning gap.

15. **Ambiguities Flagged:** Any underspecified or technically ambiguous requirement is marked `[NEEDS CLARIFICATION]` in the Requirements Coverage Summary. These are raised with the user before or during plan presentation — not silently skipped.

16. **Out of Scope Has Content:** At least one item is listed. If nothing is truly out of scope, state "None identified — all spec requirements are in scope."

### Security Boundaries

17. **No Sensitive Path Writes:** No task writes files to `.git/`, `~/.ssh/`, `node_modules/`, `.venv/`, `.env`, `.env-*`, `key.txt`, or other sensitive system paths. If a task must touch a sensitive path for legitimate reasons, the rationale is documented explicitly.

18. **Security Boundaries Explicit:** For specs that handle auth, secrets, credentials, user data, or external APIs, at least one design decision or task explicitly addresses the security boundary.

## Quick-Reference Table

| Check | Section | Fatal if Missing? |
| :--- | :--- | :--- |
| `spec:` in frontmatter | Frontmatter | Yes |
| Design decisions documented | Design Decisions | Yes |
| Interface contracts before tasks | Interface Contracts | Yes |
| Every task has `**Verify:**` | Task Breakdown | Yes |
| Requirements Coverage table | Requirements Coverage Summary | Yes |
| `[NEEDS CLARIFICATION]` for ambiguous ACs | Requirements Coverage Summary | No — but must be raised |
| `ARCHITECTURE.md` check | Constitution Check | Yes |
| No sensitive path writes | Task Breakdown | Yes |
