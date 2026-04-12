---
name: backlog
description: >
  Scan docs/jim/ for deferred and out-of-scope work, consolidate related items,
  and produce a living BACKLOG.md. Use when the user invokes /jim:backlog, wants
  to see what work has been deferred across specs, or needs a consolidated view
  of open backlog items. Do not use for prioritization (/jim:roadmap) or spec
  creation (/jim:spec).
agent: pm
---

# /jim:backlog

Scan `docs/jim/` for deferred and out-of-scope work across specs, plans, research, brainstorms, and notes. Consolidate related items, check vision alignment, and produce `docs/jim/BACKLOG.md`.

*(The `agent: pm` field in this frontmatter is a jim documentation convention, not a Claude Code routing mechanism.)*

## Process

### 1. Read strategic context

Read `docs/jim/VISION.md` if it exists. Extract the **Non-Goals** section — these are hard boundaries used for conflict checking against backlog items.

If missing, note it conversationally and proceed without vision conflict checking.

### 2. Scan structured sources

Use Glob and Grep to find deferred items in sources with predictable headings.

**Specs and plans:**
- Glob `docs/jim/specs/*/spec.md` and `docs/jim/specs/*/plan.md`
- Grep for `## Out of Scope` sections
- Read the Out of Scope section content from each matching file

**Roadmap:**
- Read `docs/jim/ROADMAP.md` if it exists
- Extract the `## Later` section — items acknowledged but not committed

### 3. Scan unstructured sources

Read these files in full — deferred items are embedded in prose without standard headings.

- Glob `docs/jim/specs/*/research.md` — look for deferred ideas, "not adopting" notes, "out of scope for" mentions
- Glob `docs/jim/brainstorms/*.md` — look for ideas that were never routed to specs
- Glob `docs/jim/notes/*.md` — look for freeform deferred ideas

Use judgment to identify deferred items in prose. There is no regex pattern that catches everything — read for intent.

### 4. Filter resolved items

Cross-reference extracted items against later specs to remove items that have since been delivered:

- Check spec `status:` frontmatter — if a spec referenced in an out-of-scope item has `status: complete`, the item may be resolved.
- Cross-reference between specs — if an out-of-scope item from an early spec names work that was pulled into a later spec's scope, and that later spec exists and is complete, the item is resolved.
- Check whether later specs or plans address the deferred item — an out-of-scope item from an early spec may have been pulled into a later spec's scope.

Err on the side of inclusion. If uncertain whether an item has been resolved, keep it in the backlog.

### 5. Check vision alignment

For each remaining item, check against VISION.md Non-Goals:

- If an item conflicts with a Non-Goal, flag it. The conflict line will appear in the output.
- If an item is compatible or neutral, no vision line is needed — compatible is the default assumption.

### 6. Consolidate related items

This is the core synthesis step. Multiple out-of-scope mentions across different specs may describe the same underlying work.

- Group items that address the same underlying concern
- Synthesize a single description that combines the unique perspectives from each source
- Preserve all source provenance links
- Do not simply list the raw mentions — write a coherent narrative for each consolidated item

The goal is actionable work items, not a raw dump of every "out of scope" bullet.

### 7. Order by relevance

Order consolidated items with broader architectural impact first, narrow tweaks last.

Items that affect more of the system's surface area, accumulate more source mentions, or have richer descriptions naturally belong higher. No explicit scoring — apply judgment.

### 8. Generate themes

Group related items into cross-cutting themes.

Each theme gets:
- A name
- A 1-2 sentence summary of what the cluster represents and why it recurs
- A list of related item titles

Themes provide **insight only** — no prescriptive recommendations. Do not suggest "consider prioritizing X" or "this should be Phase 3." The reader draws their own conclusions.

### 9. Present approval prompt

Present the proposed backlog to the user before writing. The prompt should include:

- Source scan summary (how many specs, plans, research docs, brainstorms scanned)
- Raw item count and consolidated item count
- Each consolidated item with title, one-line summary, and source list
- Identified themes with related item references

Ask: "Write this to docs/jim/BACKLOG.md?"

If the user requests changes, return to the consolidation step and adjust.

### 10. Write BACKLOG.md

After approval, read `assets/backlog-template.md` for the output structure.

Write `docs/jim/BACKLOG.md` as a complete replacement. Use Write, not Edit — each run produces a fresh document.

**Item format:**

```markdown
### {Item Title}

{Synthesized description — what the deferred work is, why it was deferred,
and what value it would deliver. Combines perspectives from all sources.}

**Sources:** `{path}`, `{path}`
**Vision conflict:** Conflicts with Non-Goal: {X}
```

The `**Vision conflict:**` line is only present when a conflict exists.

**Themes format:**

```markdown
## Themes

### {Theme Name}

{1-2 sentence summary of what this cluster represents and why it recurs.}

**Related items:** {Item Title 1}, {Item Title 2}
```

### 11. First-run guidance

If this is the first run (no existing `docs/jim/BACKLOG.md` before this invocation), inform the user:

> "This creates docs/jim/BACKLOG.md. Once this file exists, it will be automatically regenerated after each `/jim:build` completion. You can delete the file at any time to opt out of post-build updates."

### 12. Differential context

If `docs/jim/BACKLOG.md` already exists, read it before scanning. After generating the new version, briefly note what changed — new items added, items removed (resolved), items consolidated differently. This helps the user understand the delta without diffing manually.
