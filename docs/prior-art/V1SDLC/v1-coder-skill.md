---
name: coder-skill
description: Kent Beck's TDD/Tidy First methodology. Implementation gears, commit discipline, and troubleshooting. Used by @coder agent.
---

# Coder Skill

## 1. The TDD Cycle

1. **Red:** One failing test with a descriptive name. **MUST FAIL.** If it passes, the behavior already exists or the test is wrong.
2. **Green:** Write MINIMUM code to pass. "Commit sins" (constants/fakes) to reach green fast. No code beyond what the test demands.
3. **Refactor:** Eliminate duplication, normalize symmetry. One move at a time. Run tests before AND after. Apply same rigor to test files.

## 2. Implementation Gears

- **Obvious:** Solution is trivial — type it. If it fails unexpectedly, **REVERT** and downshift.
- **Fake It:** Return constant → variable → logic.
- **Triangulate:** Write 2+ tests to drive complex generalization. Only generalize when multiple tests demand it.

## 3. Tidy First

**Never mix structural and behavioral changes in one commit.**

| Category | Examples | Constraint |
|:---------|:---------|:-----------|
| **Structural** | Rename, extract, reorder, normalize | No behavior change; tests pass identically |
| **Behavioral** | Feature, fix, output change, error handling | Must have a new or modified test |

Tidy first if it makes the behavioral change easier. Otherwise, build behavior first and tidy after.

## 4. Type-Specific TDD

**Bug fix:** The Red test is the reproduction case — it MUST fail before the fix and pass after. This test becomes the permanent regression guard.

**Refactor:** No Red phase needed. Existing tests must stay green throughout. Apply one structural move at a time; re-run all tests between moves.

## 5. Commit Discipline

Commit only when: all tests pass, linter clean, single logical unit.

Prefixes: `test:`, `feat:`, `refactor:`, `fix:`

## 6. Troubleshooting

- **Unexpected Red Bar:** Step too large. **REVERT** to last commit, take a smaller "faker" step.
- **Test won't fail (Red):** Behavior may already exist. Check existing code. If so, mark task done.
- **Can't pass simply (Green):** May need a structural change first. Tidy, commit, retry.
- **Refactor breaks tests:** Undo. Make a smaller move. Match existing codebase patterns.
- **Don't:** Write multiple tests at once, refactor while red, optimize early, or touch unrelated code.

## 7. Execution Protocol

1. Read next `[ ]` task in `plan.md` + its **Verify** command.
2. Execute Red-Green-Refactor for that specific task.
3. Run ALL tests + Verify command. If pass, mark `[x]` in `plan.md`.
