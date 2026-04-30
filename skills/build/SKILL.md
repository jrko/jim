---
name: build
description: >
  Instructs the @coder to implement a spec from an approved plan using TDD.
  Use when the user invokes /jim:build or asks to implement, execute, or build
  from a spec or plan. Do not use for spec creation (/jim:spec), research
  (/jim:research), or planning (/jim:plan).
agent: coder
argument-hint: "[spec-directory-path]"
---

# /jim:build

Execute an approved plan task-by-task using Red-Green-Refactor and Tidy First commit discipline. One task at a time, tests before code, no extras beyond the plan.

First check `.jim/skills/build/references/tdd-guide.md` — if it exists, use it instead of the built-in. See `references/tdd-guide.md` for detailed methodology (TDD cycle, implementation gears, Tidy First, type-specific TDD, commit discipline, troubleshooting).

## Argument Routing

Use `$ARGUMENTS` to determine the spec directory:

| Input | Behavior |
| :--- | :--- |
| Empty | Ask: "Which spec should I build? Provide the path to the spec directory." |
| Directory path | Use as the spec directory — look for `plan.md`, `spec.md`, and `research.md` inside it |
| Path ending in `spec.md` or `plan.md` | Use the containing directory |

## Process

### 1. Resolve config

Follow `skills/_shared/resolve-paths.md` before proceeding. Resolve every `{path.*}`, `{specs.*}`, or `{workflow.*}` placeholder before passing it to a tool call.

### 2. Gate the plan

Read `plan.md` from the spec directory. Check frontmatter `status:`.

- `status: draft` — If config sets `workflow.require-plan-approval: false`, proceed despite draft status. Otherwise (default: `true`), stop. Tell the user: "This plan is still in draft. Approve it first, then re-run /jim:build."
- `status: approved` or `status: complete` with unchecked tasks — Continue.

If `plan.md` is missing, stop and tell the user: "No plan.md found in [path]. Run /jim:plan first."

If config sets `workflow.require-security: true`, check for `security.md` in the spec directory. If missing, stop and inform the user: "Security review is required before building. Run `/jim:sec` first." Otherwise (default: `false`), proceed without checking.

### 3. Load context

Read `spec.md` and `research.md` from the same directory. These provide the intent and constraints behind each task — the plan tells you *what*, the spec and research tell you *why*. Note the spec type (`feature`, `bug`, or `refactor`) — it governs Red phase behavior.

If the plan is ambiguous (a task's intent is unclear or its Verify command is malformed), STOP. Report the ambiguous task and what's unclear. Wait for the human to update the plan before continuing.

### 4. Execute the TDD loop

For each unchecked `[ ]` task in `plan.md`, in order:

**Red phase**
- Write one failing test for the specific behavior this task requires.
- Run the test via Bash. Show the output.
- Do NOT proceed to Green until test failure is confirmed via Bash output.
- If the test passes immediately: STOP. Report which task, that the test passed unexpectedly on Red, and suggest: (a) the behavior may already exist, (b) the test may be wrong. Wait for human guidance.

**Green phase**
- Write the minimum code to make the failing test pass. Fake it if needed.
- Run all tests via Bash. Show the output.
- If tests do not pass: retry with a smaller or different implementation.
- After 3 consecutive failed Green attempts on the same task: STOP. Report which task, each attempt made, what failed, and suggest: (a) update the plan, (b) run /jim:debug, or (c) adjust the approach. Wait for human guidance.

**Tidy phase**
- Refactor production and test code. One structural move at a time.
- Run all tests via Bash after each move before making the next.
- If any tidy move breaks tests: revert that move immediately. Do not fix broken tests during tidy — that is a behavioral change and belongs in a new task.

**Commit**
- Resolve `hooks.pre-commit` via Bash before committing:

  ```bash
  HOOK="$({jim_path} hooks.pre-commit)" || { echo "jim_path failed; aborting commit" >&2; exit 1; }
  [ -n "$HOOK" ] && bash -c "$HOOK"
  ```

  If `jim_path` itself exits non-zero (malformed config, schema-read failure, plugin not loaded), abort the commit and surface the error — fail-loud, not silent-skip. If the configured hook exits non-zero: show the error output, fix the issues, re-run all tests, and re-run the hook until it passes. Do NOT commit until the hook is green. If `hooks.pre-commit` is empty (default), the second line's `[ -n "$HOOK" ]` guard short-circuits and the build proceeds directly to commit.
- Follow Tidy First: one commit per logical unit, structural OR behavioral, never mixed.
- Use conventional prefixes: `test:` (Red), `feat:` / `fix:` (Green), `refactor:` (Tidy).
- First check `.jim/skills/build/references/tdd-guide.md` — if it exists, use it instead of the built-in. See `references/tdd-guide.md` — Commit Discipline section.

**Verify**
- Run the task's `**Verify:**` command from the plan via Bash. Show the output.
- If Verify fails: STOP. Report which task, the Verify command, and its output. Wait for human guidance.

**Mark**
- Update the task to `[x]` in `plan.md`. This is the only modification you make to plan.md or spec.md.

Then read the next unchecked task and repeat.

### 5. Type-specific behavior

**Feature:** Standard Red-Green-Refactor. Red writes a new test for new behavior.

**Bug:** Red writes a reproduction test — a test that MUST fail, confirming the defect exists. Do NOT proceed to Green until the reproduction test fails via Bash output. The reproduction test becomes the regression guard after the fix.

**Refactor:** No Red phase. Existing tests must pass before AND after each structural change. One move at a time; re-run all tests between moves. If any move breaks tests, revert immediately.

### 6. Completion gate

After all tasks are marked `[x]`:

1. Check if `{path.architecture}` exists. If it does, invoke `/jim:arch` to run a differential update — the architect will scan the codebase, compare against the existing document, and present any changes for your approval. If it does not exist, skip this step.
2. Check if `{path.backlog}` exists. If it does, invoke `/jim:backlog` to regenerate it — the PM will scan for deferred work, consolidate items, and present the updated backlog for your approval. If it does not exist, skip this step.
3. Report to the user and ask: "Should I mark the plan status as `complete`?"
4. STOP. Wait for the human to confirm. Do not proceed to the next SDLC phase, do not auto-invoke review. Update the plan frontmatter to `status: complete` only after explicit confirmation.

## Scope Discipline

- Do NOT add functionality, error handling, or optimizations beyond what the plan tasks specify.
- Do NOT modify `spec.md` or `plan.md` content — the only allowed change is marking tasks `[x]`.
- Do NOT proceed to the next SDLC phase (no auto-review, no auto-ship).
- If stuck and none of the above STOP conditions apply: STOP anyway. Report the task, what was attempted, and what is blocking. The human decides.

## Validation Checklist

Before beginning execution, confirm:

- [ ] `plan.md` exists and `status:` is not `draft`
- [ ] `spec.md` and `research.md` were read for context
- [ ] Spec type is noted (feature / bug / refactor)
- [ ] No ambiguous tasks in the unchecked portion of the plan
