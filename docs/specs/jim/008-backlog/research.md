---
spec: "docs/specs/jim/008-backlog/spec.md"
status: Active
date: "2026-03-31"
---

# Research: Backlog Skill

## Anchors

- **`skills/build/SKILL.md:89-95`** — Completion gate where `/jim:arch` is invoked post-build. The backlog feedback loop integrates here as an additional step.
- **`docs/specs/jim/007-arch-feedback/spec.md:28-36`** — Data flow for the arch feedback pattern. The backlog feedback loop mirrors this: check if file exists → invoke skill → present changes → continue.
- **`docs/specs/jim/007-arch-feedback/plan.md:1-101`** — Implementation plan for the arch feedback loop. Single-task plan that modified only `skills/build/SKILL.md`.
- **`agents/pm.md:38-39`** — PM agent skill list (`[spec, vision, roadmap, brainstorm]`) and tools (`[Read, Write, Edit, Glob, Grep, Agent(researcher)]`). Must add `backlog` to skills list.
- **`skills/brainstorm/SKILL.md`** — Simplest PM skill. Good structural template for the backlog skill — lightweight frontmatter, numbered steps, no assets or references needed.
- **`skills/roadmap/SKILL.md:31-34`** — Scans `docs/specs/jim/**/*.md` via Glob and Greps frontmatter `title:` fields. Closest existing pattern to the backlog skill's scanning needs.
- **`VISION.md:59-66`** — Non-Goals section. The backlog skill cross-references items against these to detect conflicts.

## Local Patterns

### Feedback loop integration

The arch feedback pattern in `skills/build/SKILL.md:89-95` is a 3-step completion gate:

1. Check if `ARCHITECTURE.md` exists → invoke `/jim:arch` differential update
2. Report to user, ask to mark plan complete
3. STOP and wait for confirmation

The backlog step would slot in after step 1, before the completion report. Same conditional pattern: check if `BACKLOG.md` exists, invoke `/jim:backlog`, present changes. The build skill invokes `/jim:arch` directly (not via a hook or subagent) — the backlog invocation follows the same direct-invocation approach.

### Skill structure convention

All PM skills follow the same pattern (`skills/spec/SKILL.md`, `skills/vision/SKILL.md`, `skills/brainstorm/SKILL.md`):

```yaml
name: {name}
description: {one-line}
agent: pm
argument-hint: "[optional]"
```

Body: numbered steps with `### N. Step name`. Validation checklist at the end. Templates in `assets/` when output has a fixed structure — the backlog skill likely needs one for the BACKLOG.md format.

### Scanning approach

The roadmap skill scans specs via `Glob docs/specs/jim/**/*.md` + `Grep title:` for frontmatter. The backlog skill needs a broader scan:

| Source | Glob pattern | What to extract |
|--------|-------------|-----------------|
| Spec out-of-scope | `docs/specs/jim/*/spec.md` | `## Out of Scope` sections |
| Plan out-of-scope | `docs/specs/jim/*/plan.md` | `## Out of Scope` sections |
| Research deferrals | `docs/specs/jim/*/research.md` | Deferred/not-adopting notes (no standard heading) |
| Brainstorms | `docs/brainstorms/*.md` | Uncaptured ideas not yet routed to specs |
| Notes | `docs/notes/*.md` | Freeform deferred ideas |
| Roadmap Later | `ROADMAP.md` | `## Later` section |
| Vision Non-Goals | `VISION.md` | `## Non-Goals` (for conflict checking, not extraction) |

Research files are the trickiest source — deferred items don't have a standard section heading. They appear as inline notes like "out of scope for v1" or "not adopting" within prose. The skill will need to read full research files and identify deferred items by content, not by heading.

### Resolved-item detection

To filter out items that have since been delivered, the skill needs to:

1. Check spec `status:` frontmatter — if a spec referenced in an out-of-scope item has `status: complete`, the item may be resolved.
2. Cross-reference between specs — 002-pm-core's out-of-scope says "strategic skills in 003-pm-strategy." If 003's spec exists and is complete, that item is resolved.
3. Check whether later specs or plans address the deferred item — an out-of-scope item from an early spec may have been pulled into a later spec's scope.

This is heuristic, not deterministic. The skill should err on the side of including uncertain items rather than silently filtering them.

### No test template

Jim is a pure-markdown plugin with no executable code and no test framework (`ARCHITECTURE.md:198-201`). Validation happens through validation checklists embedded in skills, not automated tests.

## Security & Performance

- **No security concerns.** The skill only reads markdown files under docs/ and writes one output file. No external data, no user input beyond approval, no Bash execution.
- **Performance consideration:** The skill reads many files (all specs, plans, research docs, brainstorms). For projects with a large spec archive this could be significant context. The skill should use Glob + targeted Grep rather than reading every file in full — read full content only for files where deferred items are found.

## Recommendations

### 1. Mirror the arch feedback pattern exactly

The backlog feedback loop should follow the same integration pattern as 007-arch-feedback:
- Same conditional check (file exists → invoke) — file existence is the opt-in signal
- Same direct invocation (not hook-based)
- Same approval gate (skill's own approval flow)
- Only modification to `skills/build/SKILL.md` — no changes to the backlog skill itself for post-build behavior

Running `/jim:backlog` manually is the opt-in gesture — it creates `BACKLOG.md`, and post-build updates happen from then on. Deleting the file opts out. The skill should mention this on first run so users understand the token cost implication.

**Trade-off:** This means the backlog skill doesn't know whether it's being invoked standalone or post-build. That's a feature — the skill behaves identically in both contexts.

### 2. Backlog template in assets/

A `skills/backlog/assets/backlog-template.md` would enforce the output structure (header with date, items section, themes section). This follows the convention of `skills/vision/assets/vision-template.md` and `skills/roadmap/assets/roadmap-template.md`.

**Trade-off:** The template needs to be flexible enough that the LLM can vary item count and theme count without fighting the template structure. A skeletal template with section markers (not item-level structure) is the right granularity.

### 3. Agent update is minimal

Adding `backlog` to `agents/pm.md` skills list (`line 38`) is the only agent change needed. No new tools required — the PM already has Read, Write, Edit, Glob, Grep.

### 4. Research files need content-based extraction

Unlike specs and plans where "Out of Scope" is a standard heading, research files embed deferred items in prose. The skill should read full research files and use LLM judgment to identify deferred items. This is where the "trust the LLM" principle from the brainstorm applies — no regex pattern will catch all deferred items in research prose.

### 5. Ordering approach

The spec says "broader architectural impact first, narrow tweaks last." Since the skill does a complete replacement each run, ordering is a synthesis judgment. Items with more source mentions and broader system impact naturally float up. No explicit scoring system needed — the LLM applies the ordering principle during generation.
