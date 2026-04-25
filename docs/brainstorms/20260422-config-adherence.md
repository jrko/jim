# Brainstorm: Config Adherence

*2026-04-22*

## The failure mode

Skill bodies mention default filenames inline (e.g. `ARCHITECTURE.md`) with configurability as a parenthetical hedge. Agents anchor on the concrete string and skim the parenthetical. Config resolution becomes a per-call judgment and fails under attention load — especially at the tail end of long runs.

Observed: `/jim:build` completion gate read `.jim/config.md` correctly for `path.backlog` (because `/jim:backlog` was invoked as a subskill and re-read config), but silently skipped resolution for `path.architecture` and looked at the project-root default.

## Candidate improvements

1. **Placeholders in skill bodies** — `{path.architecture}` etc., plus a standing "resolve before any filesystem call" instruction. Default filenames live in config docs only. Highest leverage, cheapest — search-and-replace across existing skills, no new infra.

2. **Shared resolve-paths preamble** — `skills/_shared/resolve-paths.md`, referenced from every skill's step 1. Agent emits a resolved-paths table before proceeding:

   ```
   Resolved paths (from .jim/config.md):
   - path.architecture: docs/jim/ARCHITECTURE.md
   - path.backlog:      docs/jim/BACKLOG.md
   ```

   Makes the override visible at the moment it matters. Belt-and-suspenders with (1).

3. **Split `/jim:build` completion gate into `/jim:complete`** — extract the ARCH update / BACKLOG regen / plan status trio out of the build tail where attention is thinnest. The bundle invites shortcuts; a dedicated skill re-reads config cleanly via the shared preamble.

4. **`jim_path` helper** — `jim_path architecture` prints the resolved path. Skills call `test -f "$(jim_path architecture)"` instead of `test -f ARCHITECTURE.md`. Makes wrong-path calls syntactically impossible in any skill that uses it. Largest investment — a new shipping component worth it only if used widely.

## How the four improvements relate

They operate at different layers, because the tool surface is heterogeneous:

- **Bash calls** can route through a helper (`$(jim_path architecture)`).
- **Native tool calls** (Write, Edit, Read, Glob) take literal path strings — no shell substitution possible. Always prose-driven.

Cross-check:

- (4) partially obviates (1) *for shell calls only*. Native tool calls still need placeholders + resolution discipline, so (1) remains load-bearing.
- (1) alone is weak without (2) — placeholders in prose with no forcing function is just a different string to skim.
- (2) alone is weak without (1) — the table exists but skill bodies still reference `ARCHITECTURE.md` inline and the agent anchors on the inline string.
- (3) is orthogonal. It addresses *where* attention fails (tail of long runs), not *how* paths resolve. If (1)+(2) are airtight, (3) contributes nothing to config adherence specifically — but see its standalone case below.

Reliability scales with how much judgment is replaced with mechanism:

| Layer | Mechanism strength |
|---|---|
| Prose instruction (status quo) | None — pure judgment |
| Placeholders alone (1) | Weak — different string, same judgment |
| Preamble + table (2) | Medium — forcing function produces a checkable artifact |
| Helper (4) | Strong — wrong path becomes syntactically impossible |

Most reliable *complete* solution: **(1) + (2) + (4)**. (2) is the forcing function, (1) ensures every reference routes through it, (4) hardens the shell surface deterministically.

## Reframing (3): it's not primarily a config fix

Config adherence motivated (3), but the stronger case is architectural. The completion gate is a distinct responsibility that happens to live in the wrong place:

- **Re-invokability.** Today completion work only runs as the tail of `/jim:build`. Interrupted builds, manual commits, post-hoc drift — no clean way to run "just the completion work." `/jim:complete` becomes a command invokable against any finished spec.
- **Separation of concerns.** `/jim:build` collapses to "execute the plan via TDD." `/jim:complete` owns "sync strategic docs." Neither needs to know the other's semantics.
- **Extension point.** Future completion concerns (changelog updates, doc-site regen, notifications, post-build review hooks) have an obvious home. Without the split, every new concern bolts onto the build tail and worsens attention decay.
- **User-visible gate.** Becomes a named checkpoint the user can defer, skip, or re-run deliberately. Today it's buried inline.
- **Failure recovery.** If ARCH update fails mid-gate today, re-entry is awkward. A discrete skill has a clean re-entry point.
- **Context hygiene** (weaker). A subskill may execute against a less-cluttered context than the tail of a long build transcript.

Even if (1)+(2) solved config adherence tomorrow, (3) still stands on these grounds. Priority raised accordingly — it's not "maybe if drift persists."

## Considered: generic `/jim:gate`

Rejected as too generalized. "Gate" means different things at different points:

- **Spec/plan approval gates** — read artifact, check frontmatter status, prompt if not approved. Readiness for next phase.
- **Task completion gate** (mid-build) — tests pass + commit made + checkbox flipped + pre-commit ran. Task hygiene.
- **Build completion gate** — update ARCH, regen BACKLOG, confirm plan status. Strategic-doc sync.

Spec and plan gates share a shape. The task gate and completion gate are doing fundamentally different work. Collapsing all into one skill becomes a switch statement on an arg — concatenation with a prefix, not generalization.

The real unit of reuse is one level down: shared *primitives* — resolve paths from config, check a frontmatter status, prompt for approval, verify file exists. Those live in `skills/_shared/` as referenced preambles and compose into focused, named gate skills. Named > arg-dispatched for a slash-command surface.

If anything, the spec/plan approval gates (which genuinely share a shape) could share an approval-gate primitive — but that's improvement (2) done well, not a new top-level skill.

## Adjacent improvements worth capturing

### 5. Config schema validation

Silent fallback is a hidden failure mode. A typo in `.jim/config.md` (`path.architechture`) makes the resolver silently return the default — user thinks they've overridden; nothing applied. Schema check at resolution time: emit an error on unknown keys, not a silent fallback. Cheap; pairs with (2).

### 6. Idempotency as a design constraint on `/jim:complete`

If the skill is re-invokable (the main non-config benefit of (3)), running it twice must be safe. Constrains how ARCH and BACKLOG get updated — regenerate not append, merge not duplicate. Worth naming up front so the implementation doesn't foreclose re-invokability.

### 7. Self-test meta-skill

Jim self-hosts, so jim can test itself. A fixture project with non-default config (e.g. `path.architecture: docs/jim/ARCHITECTURE.md`) and a meta-skill that exercises every skill against it, asserting no literal default filename appears in a filesystem call. Catches the exact bug class that motivated this brainstorm and keeps catching it as skills evolve. Strongest long-term investment — the forcing function for all the other improvements.

### 8. Resolved-paths audit log

The table from (2) lives in the conversation transcript — ephemeral. Writing it (or a compact per-invocation record) to `.jim/logs/resolved-paths.jsonl` gives a cheap audit trail for "which config was in effect when spec 007 was built?" Helps diagnose past drift. Trivial once the resolver exists.

## Design principle: prose → mechanism

Naming the pattern that runs through all of the above. Every improvement here is a move from "the agent will remember to do X" toward a mechanism that removes the remembering:

- Placeholders force the resolution question.
- The resolved-paths table forces resolution to be explicit and checkable.
- The helper removes the possibility of a wrong path at the tool boundary.
- Shared primitives remove per-skill discretion over how gates compose.
- The self-test skill removes trust-based assertions about correctness.

Generalizes beyond config adherence. Any skill step that reads "the agent should check/remember/verify" is a candidate for the same treatment. Worth keeping as an explicit lens when reviewing skills.

## Ordering

**Do now:**
- (1) Placeholders — cheapest, targets the anchor directly.
- (2) Shared resolve-paths preamble — same pass as (1).
- (5) Schema validation — cheap, belongs with (2).
- (3) Extract `/jim:complete` — justified by the standalone architectural benefits, not just drift.

**Soon after:**
- (6) Idempotency pass on `/jim:complete` — part of implementing (3) well.
- (8) Resolved-paths audit log — trivial once the resolver exists.

**When the skill surface stabilizes:**
- (4) `jim_path` helper — largest investment, clearest long-term win for Bash surface.
- (7) Self-test meta-skill — forcing function for the whole set.
