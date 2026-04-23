# Jim Configuration

Jim reads `.jim/config.md` from the project root on every skill and agent invocation. The file is optional; any omitted key uses the default, so jim works zero-config out of the box.

Configure via `/jim:config` or edit `.jim/config.md` directly. Only the YAML frontmatter is read — the prose body is for human reference.

## Paths

Strategic documents (relative to project root):

| Key | Default | Description |
|-----|---------|-------------|
| `path.vision` | `VISION.md` | Product vision document |
| `path.architecture` | `ARCHITECTURE.md` | Architecture document |
| `path.roadmap` | `ROADMAP.md` | Roadmap document |
| `path.workflow` | `WORKFLOW.md` | Workflow process document |
| `path.backlog` | `BACKLOG.md` | Backlog document |

Artifact directories (relative to project root):

| Key | Default | Description |
|-----|---------|-------------|
| `path.specs` | `docs/specs` | Spec directories (`{group}/NNN-name/`) |
| `path.brainstorms` | `docs/brainstorms` | Brainstorm files (`YYYYMMDD-topic.md`) |
| `path.debug` | `docs/debug` | Debug reports (`YYYYMMDD-topic.md`) |
| `path.research` | `docs/research` | Standalone research (non-spec-linked) |

## Spec ID format

| Key | Default | Description |
|-----|---------|-------------|
| `specs.id-padding` | `3` | Zero-padded width (3 = `001`, 4 = `0001`) |
| `specs.id-prefix` | `""` | Optional prefix (e.g., `FE-` produces `FE-001`) |

## Workflow gates

| Key | Default | Description |
|-----|---------|-------------|
| `workflow.require-research` | `false` | Plan phase stops if `research.md` is missing |
| `workflow.require-security` | `false` | Build phase stops if `security.md` is missing |
| `workflow.require-plan-approval` | `true` | Build phase stops if plan `status:` is not `approved` |

## Overlay directory

Place custom templates or reference docs under `.jim/skills/{skill}/` to override built-in plugin files — file presence wins, no config key required:

```
.jim/
  config.md
  skills/
    spec/assets/spec-template.md        # overrides built-in spec template
    build/references/tdd-guide.md       # overrides built-in TDD guide
```

Only `assets/` and `references/` are overlayable. SKILL.md and agent definitions are not. Agent overrides use Claude Code's native `.claude/agents/` mechanism.

## Example

A minimal `.jim/config.md` for a project that keeps specs under `docs/jim/specs/` and enforces security review before building:

```markdown
---
path:
  specs: docs/jim/specs
workflow:
  require-security: true
---
```

All other keys fall back to their defaults.
