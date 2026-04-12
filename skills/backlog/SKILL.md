---
name: backlog
description: >
  Scan docs/jim/ for deferred and out-of-scope work, consolidate related items,
  and produce a living BACKLOG.md; use `add <description>` to append an ad-hoc
  item without a full rescan. Use when the user invokes /jim:backlog, wants to
  see what work has been deferred across specs, or needs a consolidated view of
  open backlog items. Do not use for prioritization (/jim:roadmap) or spec
  creation (/jim:spec).
agent: pm
argument-hint: "[add <description>]"
---

# /jim:backlog

Scan `docs/jim/` for deferred and out-of-scope work across specs, plans, research, brainstorms, and notes. Consolidate related items, check vision alignment, and produce `docs/jim/BACKLOG.md`.

*(The `agent: pm` field in this frontmatter is a jim documentation convention, not a Claude Code routing mechanism.)*

## Argument Routing

Inspect `$ARGUMENTS` before entering a mode.

| `$ARGUMENTS` shape | Mode | Behavior |
| :--- | :--- | :--- |
| Empty | Regeneration | Full source scan → consolidate → preserve Ad-hoc → approval → write |
| `add <description…>` (first token literally `add`) | Append | Read `BACKLOG.md` → synthesize title → dedupe check → preview → confirm → write |
| Anything else | Regeneration | Same as empty; non-`add` arguments are ignored with a warning line in the approval prompt |

## Process — Regeneration Mode

This mode runs when no arguments are passed (or non-`add` arguments are passed). It performs a full source scan, preserves the existing Ad-hoc section, and writes a complete replacement `BACKLOG.md`.

### 1. Read strategic context

Read `docs/jim/VISION.md` if it exists. Extract the **Non-Goals** section — these are hard boundaries used for conflict checking against backlog items.

If missing, note it conversationally and proceed without vision conflict checking.

### 2. Read and extract Ad-hoc block

Read the existing `BACKLOG.md` if it exists. Locate the `## Ad-hoc` heading and extract everything between it and the next `## ` heading (typically `## Themes`), or to end of file if no subsequent `## ` heading exists. Parse `### Title` sub-blocks within that range as individual ad-hoc items.

**Abort condition:** If `BACKLOG.md` contains `### ` item blocks that appear outside any `## ` section AND no `## Ad-hoc` heading exists, abort with this warning and do not proceed:

> ! BACKLOG.md contains `### ...` item blocks outside any recognized section,
>   and no `## Ad-hoc` heading was found. This usually means the heading was
>   deleted by accident.
>
>   Please restore the `## Ad-hoc` heading above the orphaned items (or delete
>   them if intentional) and re-run /jim:backlog.

If `BACKLOG.md` does not exist, proceed with an empty ad-hoc item set.

### 3. Scan structured sources

Use Glob and Grep to find deferred items in sources with predictable headings.

**Specs and plans:**
- Glob `docs/jim/specs/*/spec.md` and `docs/jim/specs/*/plan.md`
- Grep for `## Out of Scope` sections
- Read the Out of Scope section content from each matching file

**Security reviews:**
- Glob `docs/jim/specs/*/security.md`
- Grep for findings with `**Route:** Backlog`
- Read the matching finding blocks (title, severity, description, suggestion)

**Roadmap:**
- Read `docs/jim/ROADMAP.md` if it exists
- Extract the `## Later` section — items acknowledged but not committed

### 4. Scan unstructured sources

Read these files in full — deferred items are embedded in prose without standard headings.

- Glob `docs/jim/specs/*/research.md` — look for deferred ideas, "not adopting" notes, "out of scope for" mentions
- Glob `docs/jim/brainstorms/*.md` — look for ideas that were never routed to specs
- Glob `docs/jim/notes/*.md` — look for freeform deferred ideas

Use judgment to identify deferred items in prose. There is no regex pattern that catches everything — read for intent.

### 5. Filter resolved items

Cross-reference extracted items against later specs to remove items that have since been delivered:

- Check spec `status:` frontmatter — if a spec referenced in an out-of-scope item has `status: complete`, the item may be resolved.
- Cross-reference between specs — if an out-of-scope item from an early spec names work that was pulled into a later spec's scope, and that later spec exists and is complete, the item is resolved.
- Check whether later specs or plans address the deferred item — an out-of-scope item from an early spec may have been pulled into a later spec's scope.

Err on the side of inclusion. If uncertain whether an item has been resolved, keep it in the backlog.

### 6. Check vision alignment

For each remaining sourced item AND each ad-hoc item extracted in step 2, check against VISION.md Non-Goals:

- If an item conflicts with a Non-Goal, flag it. The conflict line will appear in the output as a `**Vision conflict:**` line inside that item's block.
- If an item is compatible or neutral, no vision line is needed — compatible is the default assumption.
- Ad-hoc items receive the same vision conflict treatment as sourced items.

### 7. Consolidate related items

This is the core synthesis step. Multiple out-of-scope mentions across different specs may describe the same underlying work.

- Group items that address the same underlying concern
- Synthesize a single description that combines the unique perspectives from each source
- Preserve all source provenance links
- Do not simply list the raw mentions — write a coherent narrative for each consolidated item

The goal is actionable work items, not a raw dump of every "out of scope" bullet.

### 8. Duplicate detection

Compare each ad-hoc item (extracted in step 2) against the consolidated sourced items from step 7. Flag any item where:
- The title matches a sourced item title (case-insensitive), or
- The description is a near-match using LLM judgment (same underlying concern, even if phrased differently)

Flagged duplicates are surfaced as warnings in the approval prompt (step 10) with three resolution options: keep, remove, or merge the ad-hoc item.

### 9. Order by relevance

Order consolidated items with broader architectural impact first, narrow tweaks last.

Items that affect more of the system's surface area, accumulate more source mentions, or have richer descriptions naturally belong higher. No explicit scoring — apply judgment.

### 10. Generate themes

Group related items into cross-cutting themes. Include ad-hoc items as candidates alongside sourced items — an ad-hoc item may belong to one or more themes.

Each theme gets:
- A name
- A 1-2 sentence summary of what the cluster represents and why it recurs
- A list of related item titles (sourced and ad-hoc items may both appear)

Themes provide **insight only** — no prescriptive recommendations. Do not suggest "consider prioritizing X" or "this should be Phase 3." The reader draws their own conclusions.

### 11. Present approval prompt

Present the proposed backlog to the user before writing. The prompt should include:

- Source scan summary (how many specs, plans, research docs, brainstorms scanned)
- Raw item count and consolidated item count
- "Carrying forward N ad-hoc items from existing BACKLOG.md" (use 0 if none)
- Each consolidated item with title, one-line summary, and source list
- Any duplicate warnings from step 8 — each flagged item listed with: keep, remove, or merge options
- Identified themes with related item references

Ask: "Write this to docs/jim/BACKLOG.md?"

If the user requests changes, return to the consolidation step and adjust.

### 12. Write BACKLOG.md

After approval, read `assets/backlog-template.md` for the output structure.

Write `docs/jim/BACKLOG.md` as a complete replacement. Use Write, not Edit — each run produces a fresh document.

**Ad-hoc section:** Re-emit the `## Ad-hoc` section in its reserved position — between the sourced items area and `## Themes`. If the extracted block (from step 2) contained `### Title` items, re-emit them verbatim. If the block was empty (no `### ` items), emit the placeholder comment from the template unchanged. Whitespace normalization is permitted, but no user-authored item title or description may be lost.

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

### 13. First-run guidance

If this is the first run (no existing `docs/jim/BACKLOG.md` before this invocation), inform the user:

> "This creates docs/jim/BACKLOG.md. Once this file exists, it will be automatically regenerated after each `/jim:build` completion. You can delete the file at any time to opt out of post-build updates."

### 14. Differential context

If `docs/jim/BACKLOG.md` already exists, read it before scanning. After generating the new version, briefly note what changed — new items added, items removed (resolved), items consolidated differently. This helps the user understand the delta without diffing manually.

## Process — Append Mode

This mode runs when `$ARGUMENTS` begins with the literal token `add`. It is a fast path — it does not load VISION.md and does not run a full source scan.

### 1. Extract description

Verify `$ARGUMENTS` starts with `add`. Extract the remainder as the item description (free-form prose). If the description is empty after `add`, ask the user for a description before proceeding.

### 2. Read existing BACKLOG.md

If `BACKLOG.md` exists, read it to extract existing ad-hoc and sourced items for deduplication. If `BACKLOG.md` does not exist, prepare a minimal skeleton in memory:

```markdown
# Backlog

*Generated by `/jim:backlog` — {date}*

---

## Ad-hoc

<!-- Items added here are preserved across /jim:backlog runs.     -->
<!-- Use `### Title` headings followed by a free-form description. -->
<!-- Add items manually or via `/jim:backlog add <description>`.   -->

---

## Themes
```

### 3. Synthesize concise title

Derive a concise `### Title` (≤ 10 words) from the description using LLM judgment. The title should capture the core concern. The description becomes the item body.

### 4. Duplicate check

Compare the candidate title and description against both existing ad-hoc items and sourced items in the file. Flag any near-match (same title case-insensitively, or same underlying concern by LLM judgment).

### 5. Show preview

Display the preview to the user:

```
Preview — this will be appended to ## Ad-hoc in BACKLOG.md:

  ### {Synthesized Title}
  {Description}

[Any duplicate warnings]

Confirm append? (y/n)
```

Wait for user confirmation. Do not write without it.

### 6. Insert into ## Ad-hoc section

On confirmation, locate the `## Ad-hoc` section in `BACKLOG.md` (or the minimal skeleton) and insert the new `### Title` block at the end of that section, before the closing `---` or `## Themes` heading.

### 7. Write BACKLOG.md

Write the updated content to `docs/jim/BACKLOG.md` using Write.

### 8. Report

Report the title of the item added and its position (e.g., "Added '### {Title}' as item N in the Ad-hoc section.").
