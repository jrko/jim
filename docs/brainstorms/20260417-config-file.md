# Brainstorm: Config File Feature for Jim

*2026-04-17*

## Motivation

Jim needs to be flexible for users adopting it into existing projects. A one-size-fits-all approach annoys developers who have established conventions and preferences. The config file should let users bend jim to their project without forking or hacking skill markdown.

Broad categories in scope:
- **Project-level conventions** — artifact paths, spec grouping, ID formats, branch naming
- **Workflow preferences** — required phases, gate behavior, default models
- **Template overrides** — custom spec/plan templates, extra frontmatter fields
- **Integration settings** — issue trackers, CI references, deployment environments

## Paths

The upstream project (JamSuite/jim) uses a different layout than this fork. Upstream spreads files across separate locations (some at project root, others under docs/) rather than a single namespace (everything under docs/jim/). Config needs to support both styles.

Two kinds of path config:
1. **Explicit file paths** for key uppercase documents — ARCHITECTURE.md, VISION.md, ROADMAP.md, WORKFLOW.md, BACKLOG.md. These might live at project root, under docs/, or under docs/jim/. Each should be independently configurable.
2. **Directory paths** for artifact collections — specs, brainstorms, debug reports. These are the "where does stuff go" directories.

Open question: flat vs grouped specs. Upstream may use flat numbering (001-foo, 002-bar) while this fork uses noun-based groups (jim/001-meta, jim/002-pm-core). Config could support both via a `spec-style: flat | grouped` toggle, or just let the directory structure speak for itself.

Upstream (JamSuite/jim) and this fork have **different layouts**:

**Upstream layout:**
- Strategic docs at project root: `ARCHITECTURE.md`, `VISION.md`, `ROADMAP.md`, `WORKFLOW.md`
- Specs under `docs/specs/{group}/NNN-name/` — noun-based grouping (e.g., `docs/specs/jim/001-meta/`)
- Notes/prior-art under `docs/`
- No unified `docs/jim/` namespace

**This fork (jrko/jim) layout:**
- Everything under `docs/jim/` — strategic docs, specs, brainstorms, debug, all unified
- Flat spec numbering under `docs/jim/specs/001-meta/`

**Real-world usage** (user's other project with upstream jim):
- Specs like `docs/specs/platform/002-realtime-dashboard/spec.md` — noun groups map to product domains

Config must support both patterns. Default (zero-config) should match **upstream layout** since that's what new users install.

### Spec organization
Both upstream and fork use identical noun-based grouping logic (`{specs-dir}/{group}/NNN-name/`). The only difference is the base path:
- **Upstream**: `docs/specs/` → e.g., `docs/specs/jim/001-meta/`, `docs/specs/platform/002-realtime-dashboard/`
- **Fork**: `docs/jim/specs/` → e.g., `docs/jim/specs/001-meta/`

The fork changed hardcoded paths in skills, not grouping behavior. This is exactly what config should solve — the path is the variable, the structure is stable.

### Spec ID format
Currently hardcoded as 3-digit zero-padded (001, 002, ...). Config could allow customizing:
- Padding width (e.g., 4-digit for larger projects)
- Prefix scheme (e.g., `FE-001`, `BE-001`)
- Or just the padding width, keeping it simple

## Workflow

Config should cover both sides:
- **Soft defaults** — what agents suggest/encourage (e.g., PM nudges toward research before planning)
- **Hard enforcement** — simple checklist gates. Agents refuse to proceed if a required artifact is missing (e.g., coder won't build without research.md if `require-research: true`)

Scoped to a simple checklist for now — no per-type rules (e.g., "research required for features but not bugs"). Users configure which phases are mandatory; agents check at phase entry.

Candidate workflow toggles:
- `require-research` — plan phase checks for research.md
- `require-security` — build phase checks for security.md
- `require-plan-approval` — build phase checks for `status: approved` in plan frontmatter (this is already the default behavior, but could be made optional for solo devs who want less friction)

## File Format and Location

**`.jimrc.md`** at project root. Markdown with YAML frontmatter — consistent with every other jim artifact. Agents consume it via `Read`, no special parsing needed. The body can hold prose explaining config choices (useful for teams).

Dotfile convention: signals "project config" to developers, stays out of the way, won't be confused with a jim artifact.

Zero-config default: if `.jim/` doesn't exist, all skills/agents use upstream defaults (current hardcoded paths).

## Overlay Directory (`.jim/`)

Instead of config keys pointing to template paths, `.jim/` acts as an overlay tree that mirrors the plugin's skill and agent structure. If a file exists in the overlay, it wins over the built-in default. No file? Fall back to the plugin's built-in.

### Resolution order
1. `.jim/skills/{skill}/assets/{file}` or `.jim/skills/{skill}/references/{file}`
2. `skills/{skill}/assets/{file}` or `skills/{skill}/references/{file}` (built-in)

Same for agents:
1. `.jim/agents/{name}.md`
2. `agents/{name}.md` (built-in)

### Overlay surface

**Skills — assets and references only:**
- `assets/*.md` — templates (spec-template, plan-template, architecture-template, debug-template, research-template, roadmap-template, vision-template)
- `references/*.md` — methodology docs (tdd-guide, plan-dod, research-dod, spec-types)

**Out of overlay scope:**
- `SKILL.md` — skill self-replacement is awkward (a skill reading its own replacement and switching to it). Plugin skills are namespaced (`/jim:spec`) so Claude Code's priority system doesn't apply.
- `agents/*.md` — Claude Code already supports project-level agent overrides natively via `.claude/agents/`. No jim-specific mechanism needed.

Staleness risk applies equally to all overlay files: if upstream changes a built-in, the overlay won't pick it up automatically. This is a documentation/practice concern, not a design constraint.

## Config Sketch (`.jim/config.md`)

### Frontmatter schema

```yaml
---
# Paths — strategic documents (individual files)
path:
  vision: VISION.md                          # upstream default: project root
  architecture: ARCHITECTURE.md
  roadmap: ROADMAP.md
  workflow: WORKFLOW.md
  backlog: BACKLOG.md

# Paths — artifact directories
  specs: docs/specs                           # contains {group}/NNN-name/
  brainstorms: docs/brainstorms               # contains YYYYMMDD-topic.md
  debug: docs/debug                           # contains YYYYMMDD-topic.md
  research: docs/research                     # for standalone research (non-spec-linked)

# Spec ID format
specs:
  id-padding: 3                               # zero-padded width (default: 3 → 001, 002)
  id-prefix: ""                               # optional prefix (e.g., "FE-" → FE-001)

# Workflow gates — simple checklist enforcement
workflow:
  require-research: false                     # plan phase requires research.md
  require-security: false                     # build phase requires security.md
  require-plan-approval: true                 # build phase requires plan status: approved

# NOTE: Effort and model config were explored but dropped from the spec.
# These are platform-level settings (skill/agent frontmatter, Claude Code CLI)
# that cannot be self-applied by a skill reading a config file at runtime.
---

# Project Configuration

Optional prose explaining why this project's config differs from defaults.
```

### This fork's config would be:

```yaml
---
path:
  vision: docs/jim/VISION.md
  architecture: docs/jim/ARCHITECTURE.md
  roadmap: docs/jim/ROADMAP.md
  workflow: docs/jim/WORKFLOW.md
  backlog: docs/jim/BACKLOG.md
  specs: docs/jim/specs
  brainstorms: docs/jim/brainstorms
  debug: docs/jim/debug
---
```

Only override what differs — everything else falls back to upstream defaults.

### Design notes

- All paths are relative to project root
- File paths (vision, architecture, etc.) point to specific files
- Directory paths (specs, brainstorms, etc.) point to directories
- Omitted keys use upstream defaults
- Skills resolve paths by reading config first, then falling back to hardcoded defaults
- Effort resolution: skill frontmatter `effort:` > config per-skill > config `effort.default` > Claude Code session default
- Claude Code supports `effort:` in skill frontmatter natively — config just provides project-level defaults
- Model resolution: agent frontmatter `model:` > config per-agent > config `models.default` > Claude Code session default
- Config is read once per skill/agent invocation — only the relevant keys are consulted, the rest is ignored

## Provisioning Skill: `/jim:config`

Single skill, two modes based on file presence:
- **`.jim/config.md` doesn't exist** → scaffolding mode. Asks a few questions about project layout, creates config with appropriate defaults. Can also scaffold the overlay directory if the user wants custom templates.
- **`.jim/config.md` exists** → update mode. Reads current config, walks through what the user wants to change.

Consistent with how `/jim:vision`, `/jim:roadmap`, and other jim skills handle create-or-update via file presence.

## Config Consumption

Each skill already has a "read strategic context" step that loads VISION.md, ARCHITECTURE.md, etc. Config consumption adds `.jim/config.md` to that existing step — one extra file read per skill, no new abstractions.

Investigated alternatives:
- **Plugin-level CLAUDE.md / shared preamble** — not supported by Claude Code's plugin system
- **`@` import syntax in SKILL.md** — not supported
- **Shell injection (`` !`cat` ``)** — works but breaks pure-markdown philosophy
- **Shared reference doc** — adds tokens without clear benefit; Claude doesn't need instructions to read YAML frontmatter

Decision: update each skill's existing context-reading step. Simple, fits the pattern, small per-skill change.

### What this enables
- Swap spec template for one with extra fields your team needs
- Replace tdd-guide.md with your team's TDD approach
- Add domain-specific instructions to an agent without forking
- Partial overrides — customize one file without touching others
- Discoverable — `ls .jim/` shows all customizations at a glance

### Layout example
```
.jim/
  config.md                              # project config (paths, workflow)
  skills/
    spec/
      assets/
        spec-template.md                 # custom spec template
    build/
      references/
        tdd-guide.md                     # team's TDD methodology
  agents/
    coder.md                             # domain-augmented coder agent
```
