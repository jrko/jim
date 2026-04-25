# Architecture — Jim

*Last updated: 2026-04-24 (012-config-adherence build)*

> This document is generated and maintained by `/jim:arch`. Edit via the skill to preserve consistency.

---

## Project Structure

```
jim/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest — name, version, description
├── .claude/
│   └── settings.local.json  # Local permission allowlists (WebFetch domains, etc.)
├── agents/                  # Agent definitions — one .md per agent persona
│   ├── pm.md                # @jim:pm — product manager
│   ├── architect.md         # @jim:architect — technical architect
│   ├── researcher.md        # @jim:researcher — codebase investigator
│   ├── coder.md             # @jim:coder — TDD implementer
│   ├── meta.md              # @jim:meta — plugin developer (builds jim itself)
│   └── security.md          # @jim:security — security analyst
├── skills/                  # Skill definitions — one directory per skill
│   ├── _shared/             # Plugin-contract shared primitives (not overlayable)
│   │   ├── config-schema.md     # Authoritative config schema — keys, defaults, value rules
│   │   └── resolve-paths.md     # Shared preamble — config read, validation, resolved-paths table
│   ├── spec/                # /jim:spec — collaborative spec creation
│   ├── plan/                # /jim:plan — implementation planning
│   ├── research/            # /jim:research — codebase and landscape investigation
│   ├── build/               # /jim:build — TDD red-green-refactor execution
│   ├── debug/               # /jim:debug — structured failure diagnosis
│   ├── vision/              # /jim:vision — product vision and strategy
│   ├── roadmap/             # /jim:roadmap — execution milestones
│   ├── arch/                # /jim:arch — architecture document generation
│   ├── backlog/             # /jim:backlog — deferred work consolidation
│   ├── brainstorm/          # /jim:brainstorm — freeform ideation capture
│   ├── sec/                 # /jim:sec — security analysis and threat review
│   ├── config/              # /jim:config — project configuration scaffolding
│   ├── meta-skill/          # /jim:meta-skill — create/update jim skills
│   └── meta-agent/          # /jim:meta-agent — create/update jim agents
├── docs/
│   ├── specs/               # Spec directories grouped by domain (e.g., jim/001-meta)
│   ├── brainstorms/         # Freeform ideation (YYYYMMDD-topic.md)
│   ├── prior-art/           # Reference material from other projects
│   └── notes/               # Personal development notes
├── ARCHITECTURE.md          # This file
├── VISION.md                # Product vision — problem, solution, audience, north star
├── ROADMAP.md               # Execution sequence
├── BACKLOG.md               # Consolidated deferred work
├── WORKFLOW.md              # The SDLC process definition — commands, artifacts, philosophy
├── CLAUDE.md                # Claude Code project instructions
└── README.md                # Project readme
```

## High-Level System Diagram

```mermaid
flowchart TD
    subgraph User
        U[Developer via Claude Code]
    end

    subgraph "Strategic Skills"
        VIS["/jim:vision"]
        ROAD["/jim:roadmap"]
        ARCH["/jim:arch"]
        BACKLOG["/jim:backlog"]
        BRAIN["/jim:brainstorm"]
    end

    subgraph "SDLC Skills"
        SPEC["/jim:spec"]
        RES["/jim:research"]
        PLAN["/jim:plan"]
        SEC["/jim:sec"]
        BUILD["/jim:build"]
        DEBUG["/jim:debug"]
    end

    subgraph "Meta Skills"
        MS["/jim:meta-skill"]
        MA["/jim:meta-agent"]
        CONF["/jim:config"]
    end

    subgraph Agents
        PM["@jim:pm"]
        ARCHITECT["@jim:architect"]
        RESEARCHER["@jim:researcher"]
        SECURITY["@jim:security"]
        CODER["@jim:coder"]
        META["@jim:meta"]
    end

    subgraph Artifacts
        VDOC["VISION.md"]
        RDOC["ROADMAP.md"]
        ADOC["ARCHITECTURE.md"]
        SDOC["spec.md"]
        PDOC["plan.md"]
        RESDOC["research.md"]
        CODE["Tests + Code"]
        SECDOC["security.md"]
        BLOGDOC["BACKLOG.md"]
        BDOC["brainstorm.md"]
        DDOC["debug report"]
        CONFDOC[".jim/config.md"]
    end

    U --> VIS & ROAD & ARCH & BACKLOG & BRAIN
    U --> SPEC & RES & PLAN & SEC & BUILD & DEBUG
    U --> MS & MA & CONF

    VIS & ROAD & BACKLOG & BRAIN --> PM
    ARCH --> ARCHITECT
    SPEC --> PM
    RES --> RESEARCHER
    PLAN --> ARCHITECT
    SEC --> SECURITY
    BUILD & DEBUG --> CODER
    MS & MA & CONF --> META

    PM --> VDOC & RDOC & SDOC & BLOGDOC & BDOC
    ARCHITECT --> ADOC & PDOC
    RESEARCHER --> RESDOC
    SECURITY --> SECDOC
    CODER --> CODE & DDOC
    META --> CONFDOC

    ARCHITECT -.->|spawns| RESEARCHER
    SECURITY -.->|spawns| RESEARCHER
    META -.->|delegates| PM & ARCHITECT & RESEARCHER
```

## Core Components

### Agents

Agents are markdown files (`agents/*.md`) that define personas with frontmatter metadata. Each agent declares its name, description, skill bindings, tool permissions, and model preference.

- **Purpose:** Define the persona, responsibilities, and boundaries for each specialized role in the SDLC
- **Location:** `agents/` — `pm.md` (L1–76), `architect.md` (L1–80), `researcher.md` (L1–83), `security.md` (L1–75), `coder.md` (L1–83), `meta.md` (L1–66)
- **Interfaces:** Frontmatter fields: `name`, `description`, `skills` (list), `tools` (list), `model` (string). Body contains persona instructions, context paths, core principles, process delegation, and constraints.
- **Dependencies:** Each agent references its bound skills in `skills/`. Agents may spawn other agents via the `Agent()` tool declaration (e.g., architect, PM, and security can spawn researcher; meta can spawn PM, architect, researcher). All agents read `.jim/config.md` at spawn for configurable paths, with hardcoded defaults as fallback.
- **Key Constraints:** Agents do not cross domain boundaries — PM does not write code, coder does not modify specs, researcher does not make design decisions. All agents stop after producing an artifact and wait for human approval.

### Skills

Skills are SKILL.md files inside `skills/{name}/` directories, optionally accompanied by `assets/` (templates) and `references/` (methodology docs).

- **Purpose:** Provide the detailed process instructions that agents follow when a `/jim:{verb}` command is invoked
- **Location:** `skills/` — 14 skill directories (spec, plan, research, sec, build, debug, vision, roadmap, arch, backlog, brainstorm, config, meta-skill, meta-agent), plus the `_shared/` directory holding cross-skill plugin-contract files (see Shared Primitives below).
- **Interfaces:** Frontmatter fields: `name`, `description`, `agent` (which agent runs this skill), `argument-hint`. Body contains step-by-step process, argument routing, validation checklists.
- **Dependencies:** Skills reference their `assets/` templates and `references/` docs. Skills are bound to agents via the `agent` frontmatter field (documentation convention, not runtime routing). Every skill's step 1 follows `skills/_shared/resolve-paths.md`, which reads `.jim/config.md`, validates it against `skills/_shared/config-schema.md`, and emits a resolved-paths table inline before any filesystem-touching work. Skills with assets/references check `.jim/skills/{name}/` overlay paths before falling back to built-in files.
- **Key Constraints:** SKILL.md stays under 500 lines (progressive disclosure). Templates live in `assets/`, methodology in `references/`. Configurable paths appear in skill bodies as `{path.*}` placeholders, never as literal default filenames.

### Shared Primitives

`skills/_shared/` holds cross-skill files that every skill depends on. Distinct from per-skill `assets/` and `references/`, this directory is plugin contract — not overlayable.

- **Purpose:** Provide one canonical surface for config resolution, schema authority, and the resolved-paths forcing function. Eliminates per-skill duplication of config-handling prose.
- **Location:** `skills/_shared/` — `config-schema.md` (authoritative keys, defaults, validation rules; YAML frontmatter `keys:` list plus prose Validation Rules and Overlay Boundary sections); `resolve-paths.md` (runtime preamble — read config, validate, emit resolved-paths table, halt format for validation failures, invariants).
- **Interfaces:** `config-schema.md` exposes a YAML `keys:` list (each entry: `name`, `default`, `type`) consumed by the preamble for enumeration and validation. `resolve-paths.md` is invoked by every skill's step 1 via the canonical phrase "Resolve config — follow `skills/_shared/resolve-paths.md` before proceeding." The preamble emits the resolved-paths table marker `**Resolved paths (from .jim/config.md):**` immediately before any filesystem call.
- **Dependencies:** `resolve-paths.md` reads `config-schema.md`. Every skill's step 1 references `resolve-paths.md`. `/jim:config` reads `config-schema.md` for default values and validation rules during interview-time generation.
- **Key Constraints:** `_shared/` is **not overlayable** via `.jim/skills/_shared/` — a user file there is silently ignored. `resolve-paths.md` enforces hard-error on schema validation failures with a fixed three-line format (`✗ Config validation failed.` / `✗ Key: \`{key}\`` / `✗ Reason: …`); fallback to defaults on validation failure is prohibited.

### Plugin Manifest

- **Purpose:** Declares jim as a Claude Code plugin with name, version, and metadata
- **Location:** `.claude-plugin/plugin.json` (L1–16)
- **Interfaces:** Standard Claude Code plugin JSON: `name`, `version`, `description`, `author`, `keywords`
- **Dependencies:** None — consumed by Claude Code's plugin loader
- **Key Constraints:** `name` must be `"jim"` — all skills and agents are namespaced under this

### WORKFLOW.md

- **Purpose:** Defines the entire SDLC process — command reference, artifact locations, agent-skill composition, lifecycle details, and philosophy
- **Location:** `WORKFLOW.md` (L1–431)
- **Interfaces:** Referenced by agents and skills as the canonical process definition
- **Dependencies:** None — upstream reference document
- **Key Constraints:** Single source of truth for the SDLC process. All agents and skills must be consistent with this document.

### Spec Archive

- **Purpose:** Living development artifacts — specs, research, and plans organized by group and sequential ID
- **Location:** `docs/specs/{group}/{00X}-{name}/` — currently `docs/specs/jim/001-meta/` through `011-config/`
- **Interfaces:** Each spec directory contains up to four files: `spec.md`, `research.md`, `plan.md`, `security.md`
- **Dependencies:** Produced by PM (spec), researcher (research), and architect (plan) agents
- **Key Constraints:** IDs are 3-digit zero-padded, sequential within each group. Groups are noun-based directories. Specs must be `approved` before plans can be created.

## Data Stores

| Store | Type | Location | Purpose | Owned By |
| :--- | :--- | :--- | :--- | :--- |
| Spec Archive | Markdown files | `docs/specs/` | Persistent development artifacts — specs, research, plans, security reviews | PM, Architect, Researcher, Security |
| Strategic Docs | Markdown files | Project root (`VISION.md`, `ROADMAP.md`, `ARCHITECTURE.md`) | Project-level strategy and constraints | PM, Architect |
| Backlog | Markdown file | `BACKLOG.md` | Consolidated deferred work — sourced items, user-authored ad-hoc items, cross-cutting themes | PM |
| Brainstorms | Markdown files | `docs/brainstorms/` | Freeform ideation capture | PM |
| Debug Reports | Markdown files | `docs/debug/` | Structured failure diagnosis | Coder |
| Config and Overlay | Markdown files | `.jim/config.md`, `.jim/skills/{name}/` | Project-level configuration (paths, workflow gates, spec ID format) and asset/reference overrides. Authoritative schema lives at `skills/_shared/config-schema.md` (plugin contract, not overlayable). | Meta (via `/jim:config`); all skills follow `skills/_shared/resolve-paths.md` at step 1 |

## External Integrations

| Integration | Type | Auth Method | Rate Limits | Failure Mode |
| :--- | :--- | :--- | :--- | :--- |
| Claude Code | Host platform | N/A (plugin loaded by Claude Code) | N/A | Plugin not available if not installed |
| WebFetch/WebSearch | Claude Code tools | Domain allowlist in `.claude/settings.local.json` | Subject to provider limits | Stop and ask user to fetch manually (per CLAUDE.md policy) |

## Deployment & Infrastructure

- **Runtime:** Claude Code plugin — no standalone runtime. Requires Claude Code CLI with plugin support.
- **Entry point:** `.claude-plugin/plugin.json` — Claude Code discovers and loads the plugin from this manifest
- **Configuration:** `.claude/settings.local.json` for permission allowlists. `.jim/config.md` for project-level configuration (paths, workflow gates, spec ID format). `.jim/skills/` for asset/reference overlays. All optional — zero-config defaults match upstream layout.
- **Distribution:** Git repository. Users install by cloning/adding the repo as a Claude Code plugin.
- **Environment requirements:** Claude Code CLI. No build step, no dependencies, no package manager — pure markdown.

## Security Considerations

- **Trust boundary:** All input comes from the human developer via Claude Code. Agents do not accept external input. WebFetch/WebSearch results are the only external data — handled by stopping on failure per CLAUDE.md policy. `.jim/config.md` is repo-committable, so a malicious config in a cloned repo is a supply-chain-style threat — mitigated by `config-schema.md` value constraints (see below).
- **Secrets management:** No secrets are stored or managed. `.claude/settings.local.json` contains domain allowlists only.
- **File system access:** Agents declare tool permissions in frontmatter. Coder agent has Bash access. All agents are prohibited from writing to `.git/`, `~/.ssh/`, `node_modules/`, `.venv/`, `.env`, `.env-*`. The `.gitignore` excludes `docs/prior-art/github.com/` (downloaded references) and `Z_*` files (personal notes).
- **Security-relevant files:** `skills/_shared/config-schema.md` and `skills/_shared/resolve-paths.md` are load-bearing for config-mediated filesystem safety. **Schema invariants** (config-schema.md): `path.*` values must be relative, must not contain `..` segments, must not begin with `/`, and must resolve within the project root after following symlinks (realpath semantics); `boolean` values must be literal `true`/`false` (case variants and YAML 1.1 `yes`/`no`/`on`/`off` are rejected); unknown keys hard-error. **Preamble invariants** (resolve-paths.md): emit-before-touch (the resolved-paths table is inline before any filesystem tool call), no-silent-fallback (validation failures halt; defaults apply only to absent keys), every-invocation (the preamble runs at the start of every skill invocation; no caching across invocations), schema-is-the-authority (any change to the valid key set or value rules goes through `config-schema.md`). Weakening either file weakens config validation project-wide.
- **Auth:** None — the plugin runs within the user's Claude Code session with their permissions.
- **Known risks:** No automated validation that agents respect their declared tool boundaries — enforcement depends on Claude Code's agent tool declarations and the model following instructions. Static-audit pass at refactor close-out verifies current skill bodies invoke the preamble; ongoing enforcement that future skills do the same is deferred to a self-test meta-skill (tracked in `BACKLOG.md`).

## Development & Testing

- **Setup:** Clone the repository and configure it as a Claude Code plugin
- **Run tests:** No automated test suite — jim is a pure-markdown plugin with no executable code
- **Test framework:** N/A
- **Test conventions:** Jim validates its own output through validation checklists embedded in each skill's process section
- **Linting / formatting:** N/A — markdown only. Consistency enforced by templates in `skills/*/assets/`

## Plugin Conventions

Conventions that govern how jim's agents, skills, and tools interact with Claude Code's runtime. These are easy to get wrong because some are jim-specific conventions layered on top of Claude Code mechanics.

### Naming

- **Skills:** `name` in frontmatter must match the directory name exactly (kebab-case). Enforced by the agentskills.io open standard.
- **Agents:** `name` in frontmatter must match the filename exactly (kebab-case, without `.md`).
- **Namespacing:** All skills appear as `/jim:{name}`, all agents as `@jim:{name}`. The `jim` prefix comes from the plugin name in `plugin.json`.

### Skill Invocation

- **Description is the trigger surface.** Skill descriptions are always in Claude's context. The full SKILL.md body loads only when the skill is invoked. Write descriptions that answer *what* and *when* — vague descriptions cause undertriggering.
- **`$ARGUMENTS` substitution.** When a user types `/jim:spec my-feature`, the string `my-feature` replaces `$ARGUMENTS` in the SKILL.md body. Skills use the `argument-hint` frontmatter field to document expected arguments.
- **The `agent:` field in skill frontmatter is a jim documentation convention, not a Claude Code routing mechanism.** Claude Code only uses `agent:` natively when paired with `context: fork`. Jim uses it as metadata to record which agent runs the skill — routing happens because the skill's instructions direct Claude to the right agent.

### Agent Invocation

- **Agent markdown body = full system prompt.** Agents receive only their markdown body plus basic environment details. They do NOT inherit the parent Claude Code system prompt or conversation context. Agent definitions must be self-contained.
- **`model` defaults to `inherit`, not `sonnet`.** Must explicitly set `model: sonnet` (or `opus`, `haiku`) in agent frontmatter — omitting it inherits the parent's model.
- **`skills` field preloads full content.** Skills listed in an agent's `skills` frontmatter are injected into the agent's context at startup. Agents do NOT inherit skills from the parent conversation.
- **Plugin agents have lowest priority (4).** Project-level `.claude/agents/` overrides plugin agents of the same name. This means users can customize or override any jim agent locally.

### Subagent Delegation

- **`Agent(name1, name2)` syntax** in the `tools` field restricts which subagents an agent can spawn. Example: `tools: [Read, Write, Edit, Glob, Grep, Agent(pm, architect, researcher)]`.
- **One level only.** Subagents cannot nest — parent → child works, parent → child → grandchild does not. This is a Claude Code platform constraint.
- **Fresh context.** Subagents start with only the prompt passed via the Agent tool, not the parent's conversation history.

### Progressive Disclosure

- **SKILL.md ≤ 500 lines.** Templates go in `assets/`, methodology docs in `references/`.
- **Agent body ≤ 800 tokens.** Keep agent definitions tight — delegate detail to preloaded skills.
- **`references/` files > 300 lines should have a ToC** at the top to help Claude find relevant sections without loading everything.

### Configuration and Overlay

- **Config resolution:** Every skill's step 1 follows `skills/_shared/resolve-paths.md`. The preamble reads `.jim/config.md`, validates it against `skills/_shared/config-schema.md`, builds a resolved map (configured value where present, schema default where absent), and emits a resolved-paths table inline before any filesystem-touching work. The table is the forcing function — placeholders may not be substituted in subsequent skill steps until it has been emitted.
- **Configurable paths:** `path.*` keys redirect where skills read strategic docs and write artifacts. Skill bodies reference these as `{path.*}` placeholders (e.g., `{path.architecture}`, `{path.specs}`); literal default filenames do not appear in skill procedural prose. All values are relative to project root and validated against the schema's path constraints.
- **Workflow gates:** `workflow.require-research`, `workflow.require-security`, `workflow.require-plan-approval` control phase-entry enforcement in plan and build skills.
- **Spec ID format:** `specs.id-padding` and `specs.id-prefix` control ID generation in the spec skill.
- **Asset/reference overlay:** Skills check `.jim/skills/{skill-name}/assets/{file}` and `.jim/skills/{skill-name}/references/{file}` before reading built-in files. File presence wins — no config key needed.
- **Shared-primitives boundary:** `skills/_shared/` is **not overlayable** via `.jim/skills/_shared/`. Schema and preamble are plugin contract; user files at the overlay path are silently ignored.
- **Agent overlay:** Handled natively by Claude Code via `.claude/agents/` (project-level agents override plugin agents by priority). Not a jim-specific mechanism.

### Anti-Patterns

These are documented failure modes from prior art research (`docs/specs/jim/001-meta/research.md`):

- **Personality Soup:** "I am an AI assistant here to help" — use direct second-person voice instead ("You are the technical architect for jim").
- **Permission Creep:** Write/Bash in a read-only agent's tool list — follow least privilege.
- **Instruction Shadowing:** Repeating rules already in CLAUDE.md — agents don't inherit CLAUDE.md, but skills that run in the main context do.
- **Duplicate Logic:** Same instructions in 3+ agents — extract to a shared skill instead.

## Glossary

| Term | Definition |
| :--- | :--- |
| Skill | A `/jim:{verb}` command defined in `skills/{name}/SKILL.md` — provides process instructions for an agent |
| Agent | A `@jim:{role}` persona defined in `agents/{name}.md` — executes one or more skills |
| Spec | A structured work definition (feature, bug, or refactor) in `docs/specs/` |
| Phase gate | A human approval checkpoint between SDLC phases (e.g., spec → plan → build) |
| Tidy First | Commit discipline where structural changes are separated from behavioral changes |
| Differential update | Reading an existing artifact before modifying it — never overwrite blindly |
| Progressive disclosure | Keeping SKILL.md concise (<500 lines) by delegating detail to `assets/` and `references/` |
| Meta | Jim developing Jim — using `@jim:meta` agent with `/jim:meta-skill` and `/jim:meta-agent` to build plugin components |
| Config | Project-level configuration in `.jim/config.md` — paths, workflow gates, spec ID format |
| Overlay | User-provided asset/reference files in `.jim/skills/{name}/` that override built-in plugin files |
| Shared primitive | A cross-skill plugin-contract file under `skills/_shared/` (e.g., the resolve-paths preamble or the config schema). Not overlayable; one canonical source per concern. |
| Resolved-paths table | The inline markdown table emitted by the resolve-paths preamble at the start of every skill invocation, showing each schema key's default and resolved value. Acts as the forcing function for config adherence. |
