---
# Add configuration overrides below. Omitted keys use upstream defaults.
# Skills only read frontmatter — the prose body below is for human reference.
---

# Jim Project Configuration

This file configures jim for your project. Only add keys you want to override — omitted keys use the upstream defaults shown below.

## Paths

Strategic documents (file paths relative to project root):

| Key | Default | Description |
|-----|---------|-------------|
| `path.vision` | `VISION.md` | Product vision document |
| `path.architecture` | `ARCHITECTURE.md` | Architecture document |
| `path.roadmap` | `ROADMAP.md` | Roadmap document |
| `path.workflow` | `WORKFLOW.md` | Workflow process document |
| `path.backlog` | `BACKLOG.md` | Backlog document |

Artifact directories (directory paths relative to project root):

| Key | Default | Description |
|-----|---------|-------------|
| `path.specs` | `docs/specs` | Spec directories (`{group}/NNN-name/`) |
| `path.brainstorms` | `docs/brainstorms` | Brainstorm files (`YYYYMMDD-topic.md`) |
| `path.debug` | `docs/debug` | Debug reports (`YYYYMMDD-topic.md`) |
| `path.research` | `docs/research` | Standalone research (non-spec-linked) |

## Spec ID Format

| Key | Default | Description |
|-----|---------|-------------|
| `specs.id-padding` | `3` | Zero-padded width (3 = `001`, 4 = `0001`) |
| `specs.id-prefix` | `""` | Optional prefix (e.g., `FE-` produces `FE-001`) |

## Workflow Gates

| Key | Default | Description |
|-----|---------|-------------|
| `workflow.require-research` | `false` | Plan phase stops if `research.md` is missing |
| `workflow.require-security` | `false` | Build phase stops if `security.md` is missing |
| `workflow.require-plan-approval` | `true` | Build phase stops if plan `status:` is not `approved` |

## Overlay Directory

Place custom assets and references under `.jim/` to override built-in plugin files. Skills check the overlay path first, then fall back to the built-in.

```
.jim/
  config.md                              # this file
  skills/
    spec/
      assets/
        spec-template.md                 # overrides built-in spec template
    build/
      references/
        tdd-guide.md                     # overrides built-in TDD guide
```

Only `assets/` and `references/` are supported as overlays. SKILL.md and agent definitions are not overlayable — use `.claude/agents/` for agent overrides (Claude Code native).
