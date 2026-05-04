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
  - name: hooks.pre-commit
    default: ""
    type: string
  - name: hooks.pre-completion
    default: ""
    type: string
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

### Hook keys

| Key | Default | Type | Purpose |
| :--- | :--- | :--- | :--- |
| `hooks.pre-commit` | `""` | string | Shell command run before each commit in `/jim:build`'s TDD loop. Empty disables. |
| `hooks.pre-completion` | `""` | string | Shell command run at the start of `/jim:build`'s completion gate, before `/jim:arch` and `/jim:backlog`. Empty disables. |

**Canonical invoker.** Configured `hooks.<event>` values are dispatched by `bin/jim_run_hook` (skill prose uses the `{jim_run_hook}` derived placeholder). Per-skill in-prose composition is no longer used — see `docs/specs/jim/016-jim-run-hook-helper/spec.md`.

**Empty-default posture.** Both hooks default to the empty string. An empty value disables the hook entirely — `/jim:build` skips the corresponding step without error. The empty default provides no automated quality signal; projects opt in by configuring a value in `.jim/config.md`.

**Embedded-quote limitation.** `bin/jim_path`'s embedded-quote handling strips only the outermost `"..."` from a configured value. Complex shell logic with pipes or nested quotes (e.g., `bash -c "a && b | c"`) does not survive round-trip through the YAML parser. Package complex commands as a script file (e.g., `./ci.sh`) and configure the script path as the hook value: `hooks.pre-completion: ./ci.sh`.

**Trust model.** `hooks.*` values are arbitrary shell commands executed by Bash with the invoking user's privileges. `.jim/config.md` is therefore a shell-execution authority equivalent to a committed script file. The schema's `string` type rule ("any string accepted") defers all validation to the user. PR reviewers must scrutinize `hooks.*` changes the same way they scrutinize script-file content — a one-line config change can introduce arbitrary code execution on next `/jim:build` invocation.

**Agent-context exposure.** Hook stdout and stderr are captured into the agent's conversation context via the Bash tool. Avoid hook commands that echo untrusted file contents, environment variables, or other attacker-controllable bytes — those bytes flow into the agent's subsequent decisions during the same `/jim:build` invocation, including the `/jim:arch` and `/jim:backlog` differential updates that fire immediately after `hooks.pre-completion`.

## Derived Placeholders

In addition to the configurable keys above, the resolve-paths preamble (`skills/_shared/resolve-paths.md`) computes a small set of derived placeholders at skill-invocation time. These are not configurable — they cannot be overridden via `.jim/config.md`, and the schema does not list them in the `keys:` frontmatter.

| Placeholder | Source | Substitution form | Purpose |
| :--- | :--- | :--- | :--- |
| `{jim_path}` | Claude Code's session-level primary working directory | `jim_path --root='<absolute-project-root>'` | Shell-mediated config-adherent path resolution for skill Bash calls. Helper discovery uses Claude Code's plugin `bin/` PATH convention; the placeholder expansion injects the absolute project root via `--root` so the helper is cd-safe. The expansion is multi-token — the documented pattern for any derived placeholder that resolves to a shell command rather than a single value. |
| `{jim_run_hook}` | Claude Code's session-level primary working directory | `jim_run_hook --root='<absolute-project-root>'` | Single-call invocation of a configured `hooks.<event>` shell command — replaces the in-prose resolve-and-exec idiom from spec 015. Helper discovery and quoting algorithm are identical to `{jim_path}`. See `bin/jim_run_hook` and `docs/specs/jim/016-jim-run-hook-helper/`. |

Skills reference derived placeholders the same way they reference configurable keys — by literal `{name}` substitution at point of use. Derived placeholders do not flow into tool calls unresolved.

## Validation Rules

- **Unknown key** (a key present in `.jim/config.md` but not listed in the frontmatter above) → hard error.
- **`file-path` and `directory-path` values:** must be relative; must not contain `..` segments after normalization; must not begin with `/`. Path normalization follows symlinks (realpath semantics). The final resolved target — after following any symlinks in the lexical path — must lie within the project root. Symlinks whose targets lie outside the project root are rejected with the `value must resolve within project root` reason, regardless of whether the lexical path contains `..`.
- **`positive-integer`:** must parse as an integer ≥ 1. Violations reject with `must be a positive integer`.
- **`boolean`:** must be the literal YAML `true` or `false`. Case variants (`True`, `FALSE`) and legacy YAML 1.1 forms (`yes`, `no`, `on`, `off`) are rejected with a `must be literal true or false` reason.
- **`string`:** any string accepted (no constraint beyond the type requirement).

Every rule violation halts execution immediately via the error format defined in `skills/_shared/resolve-paths.md`. Fallback to defaults is prohibited — validation failures must surface to the user, not silently disappear.

## Schema Format Constraint

The frontmatter of this file and the frontmatter of any `.jim/config.md` must conform to a restricted YAML subset that is parseable by `awk` without a YAML library. This constraint preserves jim's no-dependency stance and bounds the parser audit surface.

- Single YAML document (no `---` separators inside frontmatter beyond the start and end markers).
- The `keys:` value is a sequence of mappings; each mapping has exactly three fields: `name`, `default`, `type`.
- `.jim/config.md` frontmatter is a flat mapping of `key.name: value` entries (one per line).
- All scalar values are single-line. No `|`, `>`, or implicit multi-line continuation.
- No anchors (`&`), aliases (`*`), merge tags (`<<:`), or explicit type tags (`!!str`, etc.).
- Quoting is permitted for empty strings (`""`), integer-typed defaults, and boolean-typed defaults. Otherwise unquoted plain scalars are required.

Schema or config additions that violate this constraint cause `bin/jim_path` to exit 1 with `jim_path: malformed schema` or `jim_path: malformed config`. Future schema additions must remain in this restricted form.

## Overlay Directory

Place custom assets and references under `.jim/` to override built-in plugin files. Skills check the overlay path first, then fall back to the plugin file.

```
.jim/
  config.md                              # project config (this file's frontmatter)
  skills/
    spec/
      assets/
        spec-template.md                 # overrides built-in spec template
    build/
      references/
        tdd-guide.md                     # overrides built-in TDD guide
```

Only `assets/` and `references/` under a named skill are overlayable.

**Plugin contract — not overlayable:**

- `SKILL.md` files — skill bodies are plugin contract.
- Agent definitions in `agents/` — for agent overrides, use Claude Code's native `.claude/agents/` mechanism.
- `skills/_shared/` (this directory) — `config-schema.md` and `resolve-paths.md` are the schema and preamble. A user who creates `.jim/skills/_shared/config-schema.md` or `.jim/skills/_shared/resolve-paths.md` expecting to add keys, loosen validation, or swap the preamble will find that file silently ignored.
