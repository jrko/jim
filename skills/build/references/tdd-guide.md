# TDD and Tidy First Reference Guide

Evolved from `docs/jim/prior-art/V1SDLC/v1-coder-skill.md`. This guide covers the methodology behind `/jim:build`. Read it when you need detail on a specific practice — the skill covers the process, this covers the reasoning.

## Table of Contents

1. [The TDD Cycle](#1-the-tdd-cycle)
2. [Implementation Gears](#2-implementation-gears)
3. [Tidy First](#3-tidy-first)
4. [Type-Specific TDD](#4-type-specific-tdd)
5. [Commit Discipline](#5-commit-discipline)
6. [Troubleshooting](#6-troubleshooting)

---

## 1. The TDD Cycle

Red → Green → Tidy. In that order, always.

**Red:** Write one failing test with a descriptive name. Run it via Bash and confirm failure before writing any production code. Do NOT proceed to Green until test failure is confirmed via Bash output. If the test passes immediately, the behavior already exists or the test is wrong — STOP and investigate before continuing.

**Green:** Write the minimum code to make the test pass. "Commit sins" are allowed here — return a constant, hardcode a value, fake the logic. The goal is green fast, not elegant. Do not write code the test does not yet demand. Run all tests via Bash before declaring green.

**Tidy (Refactor):** Eliminate duplication, normalize symmetry, improve naming. One structural move at a time. Run tests before AND after each move. Apply the same rigor to test files — tests are production code too. Do NOT proceed to commit until tests are green after each tidy move.

---

## 2. Implementation Gears

Use these when Green is not obvious. Downshift when a higher gear fails.

**Obvious:** The solution is clear — type it. If tests fail unexpectedly, REVERT to last commit and downshift. Do not debug an obvious implementation that didn't work; a smaller step is faster.

**Fake It:** Return a constant, then a variable, then logic. Progress one step at a time. Each step earns a commit.

**Triangulate:** Write two or more tests to force generalization. Only generalize when multiple failing tests demand it — premature generalization is scope creep. This gear is correct when one test alone is satisfied by a fake.

---

## 3. Tidy First

Never mix structural and behavioral changes in one commit. This is the non-negotiable rule.

| Category | Examples | Constraint |
| :--- | :--- | :--- |
| Structural | Rename, extract method, reorder, move file, normalize | No behavior change — tests pass identically before and after |
| Behavioral | New feature, bug fix, output change, error handling | Must have a new or modified test that was red before the change |

**When to tidy:** Tidy before a behavioral change if the structure makes the change easier. Tidy after if the behavior was easier to reach with messy structure. Never tidy during — the moment you mix structural and behavioral changes in the same commit, you lose the ability to bisect failures cleanly.

**Why this matters:** A clean git history where structural and behavioral commits alternate is reviewable, bisectable, and revertable. A tangled commit where a rename and a feature land together is none of those things.

---

## 4. Type-Specific TDD

### Feature

Standard Red-Green-Refactor. Red writes a new test for new behavior the plan requires. Green implements only what that test demands. Tidy cleans up after.

### Bug Fix

Red writes a reproduction test — a test that MUST fail, confirming the defect exists in the current codebase. Do NOT proceed to Green until the reproduction test fails via Bash output. This is the most important gate in bug fix work: if the test passes before the fix, either the bug was already fixed elsewhere or the test does not reproduce the defect.

Green fixes the bug with the minimal change that makes the reproduction test pass. Do not fix related issues in the same commit.

The reproduction test becomes the permanent regression guard. It must remain in the test suite after the fix.

### Refactor

No Red phase needed — there is no new behavior. Existing tests must pass before AND after each structural change. One move at a time; re-run all tests between moves. If any move breaks tests, REVERT immediately — never fix broken tests during a refactor, because that means the refactor changed behavior.

---

## 5. Commit Discipline

Commit only when: all tests pass, linter is clean, and the commit represents a single logical unit.

**Conventional commit prefixes:**

| Prefix | Use |
| :--- | :--- |
| `test:` | Adding or modifying a test (Red phase commit) |
| `feat:` | New behavior passing (Green phase commit for features) |
| `fix:` | Bug fix passing (Green phase commit for bugs) |
| `refactor:` | Structural change only, no behavior change (Tidy phase commit) |

One commit per logical unit. A logical unit is one of: a new failing test, a minimal implementation that passes it, or a single structural move. Do not combine multiple logical units into one commit — it destroys traceability.

---

## 6. Troubleshooting

**Unexpected Red Bar (test fails when it should pass):** The step was too large. REVERT to the last commit and downshift to a smaller implementation gear. Do not attempt to debug a failing Green — revert and take a smaller step.

**Test won't fail on Red:** The behavior may already exist. Check the existing codebase. If the behavior is already implemented, mark the task done and move on. If the test itself is wrong (testing the wrong thing), fix the test first and commit it before writing production code.

**Can't reach Green simply:** A structural change may be needed before the behavioral change can land cleanly. Tidy (structural commit), then retry Green. If still stuck after three attempts, STOP — do not continue pushing on a stuck Green. Report the task, attempts, and what failed, then wait for human guidance.

**Refactor breaks tests:** UNDO the structural change immediately. Make a smaller move. Match existing codebase patterns — a refactor that breaks tests is a behavior change in disguise.

**Things that always cause problems:**
- Writing multiple tests at once before seeing any fail
- Refactoring while Red (mixing phases)
- Early optimization (performance belongs in a later plan task, not during feature work)
- Touching unrelated code (scope creep — even "obvious" improvements belong in a separate plan task)
