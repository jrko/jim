---
keys:
  - name: path.vision
    default: VISION.md
    type: file-path
  - name: path.architecture
    default: ARCHITECTURE.md
    type: file-path
  - name: path.roadmap
    default: ROADMAP.md
    type: file-path
  - name: path.workflow
    default: WORKFLOW.md
    type: file-path
  - name: path.backlog
    default: BACKLOG.md
    type: file-path
  - name: path.specs
    default: docs/specs
    type: directory-path
  - name: path.brainstorms
    default: docs/brainstorms
    type: directory-path
  - name: path.debug
    default: docs/debug
    type: directory-path
  - name: path.research
    default: docs/research
    type: directory-path
  - name: path.notes
    default: docs/notes
    type: directory-path
  - name: specs.id-padding
    default: 3
    type: positive-integer
  - name: specs.id-prefix
    default: ""
    type: string
  - name: workflow.require-research
    default: false
    type: boolean
  - name: workflow.require-security
    default: false
    type: boolean
  - name: workflow.require-plan-approval
    default: true
    type: boolean
---

# Jim Config Schema

This file is the authoritative definition of every key that is valid in a project's `.jim/config.md`. The frontmatter above enumerates every key with its default value and value type; the sections below document per-key purpose, validation rules, and the overlay boundary.

Skills read this schema at runtime via the resolve-paths preamble (`skills/_shared/resolve-paths.md`) to validate `.jim/config.md` and resolve `{path.*}`, `{specs.*}`, and `{workflow.*}` placeholders. `/jim:config` reads this schema when scaffolding or updating a project's config. Developers and users consult this file as the single source of truth for what may appear in `.jim/config.md`.

## Keys

### Path keys

All `path.*` keys are relative to the project root.

| Key | Default | Type | Purpose |
| :--- | :--- | :--- | :--- |
| `path.vision` | `VISION.md` | file-path | Strategic vision document. Read by skills that reference vision for alignment (spec, plan, research, sec, backlog, brainstorm, roadmap, vision, arch). |
| `path.architecture` | `ARCHITECTURE.md` | file-path | Architecture document. Read for locked technical constraints by spec, plan, research, sec, vision, arch, and the build completion gate. |
| `path.roadmap` | `ROADMAP.md` | file-path | Execution sequence document. Read by backlog, roadmap, and brainstorm. |
| `path.workflow` | `WORKFLOW.md` | file-path | SDLC process reference. Consumed contextually by meta, meta-skill, meta-agent. |
| `path.backlog` | `BACKLOG.md` | file-path | Consolidated backlog document. Written and read by backlog; read by the build completion gate. |
| `path.specs` | `docs/specs` | directory-path | Root for spec/plan/research/security artifacts organized by group and ID. |
| `path.brainstorms` | `docs/brainstorms` | directory-path | Freeform ideation file directory. |
| `path.debug` | `docs/debug` | directory-path | Debug report directory. |
| `path.research` | `docs/research` | directory-path | Standalone research file directory (distinct from per-spec `research.md`). |
| `path.notes` | `docs/notes` | directory-path | User-authored notes directory. Scanned by backlog for deferred items. |

### Spec ID format keys

| Key | Default | Type | Purpose |
| :--- | :--- | :--- | :--- |
| `specs.id-padding` | `3` | positive-integer | Zero-padded width for spec IDs. `3` produces `001`, `002`, …; `4` produces `0001`, etc. |
| `specs.id-prefix` | `""` | string | Optional prefix prepended to every spec ID (e.g. `FE-` produces `FE-001`). Empty string means no prefix. |

### Workflow gate keys

| Key | Default | Type | Purpose |
| :--- | :--- | :--- | :--- |
| `workflow.require-research` | `false` | boolean | When `true`, `/jim:plan` halts if no `research.md` exists in the spec directory. |
| `workflow.require-security` | `false` | boolean | When `true`, `/jim:build` halts if no `security.md` exists in the spec directory. |
| `workflow.require-plan-approval` | `true` | boolean | When `true`, `/jim:build` halts if `plan.md` is still `status: draft`. |

## Validation Rules

- **Unknown key** (a key present in `.jim/config.md` but not listed in the frontmatter above) → hard error.
- **`file-path` and `directory-path` values:** must be relative; must not contain `..` segments after normalization; must not begin with `/`. Path normalization follows symlinks (realpath semantics). The final resolved target — after following any symlinks in the lexical path — must lie within the project root. Symlinks whose targets lie outside the project root are rejected with the `value must resolve within project root` reason, regardless of whether the lexical path contains `..`.
- **`positive-integer`:** must parse as an integer ≥ 1. Violations reject with `must be a positive integer`.
- **`boolean`:** must be the literal YAML `true` or `false`. Case variants (`True`, `FALSE`) and legacy YAML 1.1 forms (`yes`, `no`, `on`, `off`) are rejected with a `must be literal true or false` reason.
- **`string`:** any string accepted (no constraint beyond the type requirement).

Every rule violation halts execution immediately via the error format defined in `skills/_shared/resolve-paths.md`. Fallback to defaults is prohibited — validation failures must surface to the user, not silently disappear.

## Overlay Boundary

`skills/_shared/` is part of the jim plugin contract and is **not** overlayable via `.jim/skills/_shared/`. A user who creates `.jim/skills/_shared/config-schema.md` or `.jim/skills/_shared/resolve-paths.md` expecting to add keys, loosen validation, or swap the preamble will find that file silently ignored.

Per-skill asset and reference overlays under `.jim/skills/{skill-name}/assets/` and `.jim/skills/{skill-name}/references/` remain supported as documented in `ARCHITECTURE.md`; the shared-primitives directory is a distinct surface and operates on plugin-contract semantics.
