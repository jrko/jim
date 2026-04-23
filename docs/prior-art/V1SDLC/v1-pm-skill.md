---
name: v1-pm-skill
description: Standards for grouped, incremental specs (features, bugs, refactors).
---

# V1 PM Skill: Spec Standards

## 1. Grouping & Pathing
- **Folders:** Group by noun (`search`, `auth`). Use broad categories.
- **IDs:** 3-digit, 0-padded (001, 002). No skipped numbers.
- **Path Pattern:** `docs/specs/{group}/{ID}-{kebab-name}/spec.md`
  - *Correct:* `docs/specs/comments/004-avatar-support/spec.md`
  - *Incorrect:* `docs/specs/comments/avatar-support.md`

## 2. Type-Specific Section Requirements

### All Types
- **Overview:** Max 2 sentences. Focus on the "What."
- **Acceptance Criteria (AC):** Binary pass/fail.
  - *Bad:* "Fast loading." *Good:* "List loads <500ms with 100 items."
- **Out of Scope:** Explicitly list excluded items to prevent scope creep.
- **Open Questions:** Track unknowns. Mark resolved with ~~strikethrough~~. Write "None" when clear.

### Feature Only
- **Problem:** User pain, not solution. (Skip if obvious like "Dark Mode").
- **User Stories:** "As a [role], I can [action] so that [benefit]." Specify role (Admin, Guest, etc).
- **Mockup:** ASCII art for UI. Remove if no UI component.
- **Data Flow:** Mermaid `flowchart`. 5-10 nodes max. Avoid HTML tags.

### Bug Only
- **Defect Profile:** Must include Steps to Reproduce, Actual Behavior, Expected Behavior.
- **AC must include:** "Regression test covers the reported scenario."

### Refactor Only
- **Refactor Rationale:** Must include Motivation, Current State, Desired State, Affected Systems.
- **AC must include:** "Existing tests pass without modification."

## 3. Status Values
- `draft`: Scoping/Interviewing.
- `approved`: Ready for @v1-architect.
- `in-progress` → `complete` → `deprecated`

## 4. Anti-Patterns
- **Kitchen Sink:** Spec does too much. (Split it).
- **Vague Criteria:** Untestable terms like "works well" or "clean."
- **Solution Masquerading:** "We need a cache" (Solution) vs "Page takes 8s" (Problem).
- **Empty Out of Scope:** If nothing is excluded, boundaries are too soft.
- **Premature Tech:** Specifying DB schemas or APIs (Leave for @v1-architect).
- **Wrong Type:** Using feature sections for a bug (symptoms: no user stories make sense, problem is "it's broken"). Switch to bug type.
