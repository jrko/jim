# Development Workflow: The "Jim" Plugin

This document defines the agentic development workflow for projects using Jim. "Jim" is a **Claude Code plugin** that provides a command-driven SDLC through namespaced skills and specialized agents. You talk to Jim like a person: `/jim:spec`, `/jim:plan`, `/jim:build`.

---

## Agentic SDLC Overview

```
[ BRAINSTORM ] <──────────────────────────────────┐
      │                                           │
      ▼                                           │
  [ SPEC ] <───┐                                  │
      │        │ (Feedback Loop)                  │ (Strategic Loop)
      ▼        │                                  │
  [ PLAN ] <───┘                                  │
      │                                           │
      ▼                                           │
  [ BUILD ] ───► [ REVIEW* ] ───► [ SHIP* ] ──────┘

  * = not yet implemented
```

### The Feedback Loop

This is not waterfall. Any downstream phase can send you back upstream:

- During **plan** you discover the spec is wrong → update the spec.
- During **build** you discover the plan missed something → update the plan.
- During **build** a failing test reveals a spec gap → update the spec, then the plan.

**No ghost tasks:** if a code change is needed that isn't in the plan, the plan must be updated before the code is touched. The artifacts are living documents, not approval gates you pass through once.

### The Research Engine

Research is not a gated phase; it is an agile service that grounds the SDLC in reality.

1. **Pre-Spec (Exploratory):** PM invokes `/jim:research` to explore libraries or codebase
   feasibility before committing to a spec.
2. **In-Plan (Autonomous):** If `/jim:plan` is run and no `research.md` exists (or it's
   insufficient), the Architect automatically spawns the Researcher.
3. **The Feedback Loop:**
   - **Research → Spec:** If research reveals a requirement is technically impossible, it
     signals the PM to update the spec.
   - **Research → Plan:** If research is updated after a plan exists, it signals the
     Architect to re-validate the implementation anchors.

---

## Command Reference

| Command | What It Does | Agent | Output |
|---------|-------------|-------|--------|
| `/jim:spec` | Define the work — feature, bug, or refactor | `@jim:pm` | `spec.md` |
| `/jim:plan` | Research codebase + break into atomic tasks | `@jim:architect` | `plan.md` |
| `/jim:research` | Investigate codebase, external docs, and technical landscape | `@jim:researcher` | `research.md` |
| `/jim:build` | TDD red-green-refactor + commit per task | `@jim:coder` | Tests + code |
| `/jim:review` | Quality and security gate *(not yet implemented)* | `@jim:reviewer` | TBD |
| `/jim:ship` | PR, deploy, update roadmap *(not yet implemented)* | TBD | Merged PR |
| `/jim:vision` | Create/update project vision and strategy | `@jim:pm` | `VISION.md` |
| `/jim:roadmap` | Create/update execution milestones and phase sequence | `@jim:pm` | `ROADMAP.md` |
| `/jim:arch` | Create/update technical architecture | `@jim:architect` | `ARCHITECTURE.md` |
| `/jim:debug` | Diagnose failures, produce report for spec/plan cycle | `@jim:coder` | `debug/{YYYYMMDD}-{topic}.md` |
| `/jim:backlog` | Scan deferred work, consolidate, and produce BACKLOG.md; use `add <desc>` to append an ad-hoc item without a full rescan | `@jim:pm` | `BACKLOG.md` |
| `/jim:brainstorm` | Freeform ideation — exploratory notes | `@jim:pm` | `brainstorms/{YYYYMMDD}-{topic}.md` |
| `/jim:meta-skill` | Create/update a jim plugin skill from spec | `@jim:meta` | `jim/skills/{name}/SKILL.md` |
| `/jim:meta-agent` | Create/update a jim plugin agent from spec | `@jim:meta` | `jim/agents/{name}.md` |

---

## Artifacts

### Project Artifacts

| Artifact | Location | What It Is | Managed By |
|----------|----------|------------|------------|
| Vision | `VISION.md` (project root) | Big picture — problem statement, value prop, target audience, competitive landscape | `/jim:vision` |
| Architecture | `ARCHITECTURE.md` (project root) | Technical foundation — codemap, system diagram, tech stack, data structures, architectural invariants | `/jim:arch` |
| Roadmap | `ROADMAP.md` (project root) | Execution sequence — milestones, phase breakdowns, links to numbered specs with status | `/jim:roadmap` |
| Spec | `docs/specs/{group}/{00X}-{name}/spec.md` | Work definition — requirements, acceptance criteria, spec type (feature/bug/refactor) | `/jim:spec` |
| Plan | `docs/specs/{group}/{00X}-{name}/plan.md` | Implementation path — codebase research, atomic tasks, dependencies | `/jim:plan` |
| Debug Report | `docs/debug/{YYYYMMDD}-{topic}.md` | Diagnosis — error analysis, root cause, references to affected specs | `/jim:debug` |
| Backlog | `BACKLOG.md` | Consolidated deferred work — out-of-scope items, unresolved ideas, cross-cutting themes | `/jim:backlog` |
| Brainstorm | `docs/brainstorms/{YYYYMMDD}-{topic}.md` | Exploratory notes — ideas, risks, options, may feed into specs | `/jim:brainstorm` |

### Plugin Artifacts (Jim developing Jim)

| Artifact | Location | What It Is | Managed By |
|----------|----------|------------|------------|
| Plugin Vision | `jim/VISION.md` | Why Jim exists, what problem it solves | `/jim:vision` (from within jim/) |
| Plugin Architecture | `jim/ARCHITECTURE.md` | Plugin structure, agent-skill composition, naming conventions | `/jim:arch` (from within jim/) |
| Plugin Roadmap | `jim/ROADMAP.md` | Which skills/agents to build in what order | `/jim:roadmap` (from within jim/) |
| Plugin Workflow | `jim/docs/workflow.md` | This file — the SDLC process itself | Manual or `/jim:meta-skill` |
| Skill | `jim/skills/{name}/SKILL.md` | A jim plugin skill — instructions, templates, references | `/jim:meta-skill` |
| Agent | `jim/agents/{name}.md` | A jim plugin agent — persona, tools, skill composition | `/jim:meta-agent` |

---

## Philosophy

**Spec-Driven Development:** We separate thinking from doing. Requirements, research, and plans are defined and approved before implementation begins.

**Strategic Alignment:** Every spec traces back to the project's vision (why), architecture (how), and roadmap (when).

**Test-Driven Development:** Following Kent Beck's TDD — one task, one failing test, minimum code to pass, tidy, commit, next. Structural changes (refactors) are always separate from behavioral changes.

**Human-in-the-Loop:** Agents STOP and wait for approval after completing each phase gate or atomic task. No autonomous multi-phase execution.

**Differential Updates:** Running any `/jim:` command on an existing artifact refines it based on new context, never overwrites blindly.

---

## Plugin Architecture

"Jim" is a Claude Code plugin. All skills are namespaced as `/jim:{verb}`, all agents as `@jim:{role}`.

### Plugin Directory

```
jim/
├── .claude-plugin/
│   └── plugin.json              # { "name": "jim", "version": "1.0.0" }
│
├── VISION.md                    # Why Jim exists (Jim's own strategic docs)
├── ARCHITECTURE.md              # How Jim is structured
├── ROADMAP.md                   # What to build next for Jim
│
├── agents/
│   ├── pm.md                    # → @jim:pm
│   ├── architect.md             # → @jim:architect
│   ├── researcher.md            # → @jim:researcher
│   ├── coder.md                 # → @jim:coder
│   ├── reviewer.md              # → @jim:reviewer
│   └── meta.md                  # → @jim:meta
│
├── skills/
│   │
│   │  # ── SDLC Lifecycle ──
│   ├── spec/
│   │   ├── SKILL.md             # → /jim:spec
│   │   ├── assets/
│   │   │   └── spec-template.md
│   │   └── references/
│   │       └── spec-types.md
│   │
│   ├── plan/
│   │   ├── SKILL.md             # → /jim:plan
│   │   └── assets/
│   │       └── plan-template.md
│   │
│   ├── research/
│   │   └── SKILL.md             # → /jim:research
│   │
│   ├── build/
│   │   ├── SKILL.md             # → /jim:build
│   │   └── references/
│   │       └── tdd-guide.md
│   │
│   ├── review/
│   │   └── SKILL.md             # → /jim:review (not yet implemented)
│   │
│   ├── ship/
│   │   └── SKILL.md             # → /jim:ship (not yet implemented)
│   │
│   │  # ── Strategic ──
│   ├── vision/
│   │   ├── SKILL.md             # → /jim:vision
│   │   └── assets/
│   │       └── vision-template.md
│   │
│   ├── roadmap/
│   │   ├── SKILL.md             # → /jim:roadmap
│   │   └── assets/
│   │       └── roadmap-template.md
│   │
│   ├── arch/
│   │   ├── SKILL.md             # → /jim:arch
│   │   └── assets/
│   │       └── architecture-template.md
│   │
│   ├── backlog/
│   │   ├── SKILL.md             # → /jim:backlog
│   │   └── assets/
│   │       └── backlog-template.md
│   │
│   │  # ── Utilities ──
│   ├── debug/
│   │   ├── SKILL.md             # → /jim:debug
│   │   └── assets/
│   │       └── debug-template.md
│   │
│   ├── brainstorm/
│   │   └── SKILL.md             # → /jim:brainstorm
│   │
│   │  # ── Meta (Jim building Jim) ──
│   ├── meta-skill/
│   │   ├── SKILL.md             # → /jim:meta-skill
│   │   └── references/
│   │       └── skill-standards.md
│   │
│   └── meta-agent/
│       ├── SKILL.md             # → /jim:meta-agent
│       └── references/
│           └── agent-standards.md
│
├── docs/
│   └── workflow.md              # This file — the SDLC process itself
│
└── README.md
```

### Agents

| Agent | Role | Used By |
|-------|------|---------|
| `@jim:pm` | Product strategy, specs, vision, roadmap, backlog | `/jim:spec`, `/jim:vision`, `/jim:roadmap`, `/jim:backlog`, `/jim:brainstorm` |
| `@jim:architect` | Technical planning, architecture | `/jim:plan`, `/jim:arch` |
| `@jim:researcher` | Codebase investigation and technical landscape research | `/jim:research`, invoked by PM or architect |
| `@jim:coder` | TDD implementation, debugging | `/jim:build`, `/jim:debug` |
| `@jim:reviewer` | Quality gate *(not yet implemented)* | TBD |
| `@jim:meta` | Plugin development — builds skills and agents | `/jim:meta-skill`, `/jim:meta-agent` |

### Agent ↔ Skill Composition

Agents declare which skills they compose in their frontmatter. Skills declare which agent runs them.

```yaml
# jim/agents/pm.md
---
name: pm
description: Product manager. Defines specs and maintains strategic alignment.
skills:
  - spec
  - vision
  - roadmap
  - brainstorm
  - backlog
---
```

```yaml
# jim/agents/architect.md
---
name: architect
description: Technical architect. Plans implementation and maintains
  the technical architecture. Delegates codebase investigation to
  @jim:researcher subagent.
skills:
  - plan
  - arch
---
```

```yaml
# jim/agents/researcher.md
---
name: researcher
description: Codebase investigator. Explores existing code patterns,
  integration points, and blast radius. Read-only. Reports findings
  back to the architect.
tools: Read, Grep, Glob
---
```

```yaml
# jim/agents/coder.md
---
name: coder
description: Developer. Builds via TDD and debugs failures.
skills:
  - build
  - debug
---
```

```yaml
# jim/agents/reviewer.md (not yet implemented)
---
name: reviewer
description: Code reviewer. Quality and security gate.
skills:
  - review
---
```

```yaml
# jim/agents/meta.md
---
name: meta
description: Plugin developer. Creates and updates jim skills and agents
  from specs. Reads workflow.md for SDLC context and enforces plugin
  conventions (frontmatter, naming, progressive disclosure).
skills:
  - meta-skill
  - meta-agent
tools: Read, Write, Edit, Glob, Grep
---
```

Skills specify their agent in frontmatter:

```yaml
# jim/skills/spec/SKILL.md
---
name: spec
description: Create, update, or validate feature/bug/refactor specs.
  Use when the user wants to scope work, write a spec, or define requirements.
agent: pm
---
```

```yaml
# jim/skills/arch/SKILL.md
---
name: arch
description: Create or update the project ARCHITECTURE.md following
  the architecture.md standard. Use when discussing tech stack, system
  design, codemap, or architectural decisions.
agent: architect
---
```

```yaml
# jim/skills/meta-skill/SKILL.md
---
name: meta-skill
description: Create or update a jim plugin skill. Reads workflow.md
  and spec to generate SKILL.md with proper frontmatter, directory
  structure (assets/, references/), and agent binding.
agent: meta
---
```

---

## The Lifecycle in Detail

### `/jim:spec`

**Purpose:** Turn a rough idea into a clear, actionable spec.

**Process:**
1. Describe your idea, bug report, or refactor motivation
2. PM determines spec type (`feature`, `bug`, `refactor`) and asks clarifying questions (1-2 at a time)
3. Checks alignment against VISION.md and ARCHITECTURE.md
4. Generates spec using template from `spec/assets/spec-template.md`
5. You review and approve or request changes

**Output:** `docs/specs/{group}/{00X}-{name}/spec.md`

**Gate:** Human approves the spec. Status changes from `draft` to `approved`.

### `/jim:plan`

**Purpose:** Research the codebase and create an implementation plan.

**Process:**
1. Reads the approved spec
2. Delegates codebase investigation to `@jim:researcher` subagent — integration points, fault locations (bugs), blast radius (refactors)
3. Reviews research findings
4. Checks ARCHITECTURE.md for architectural constraints
5. Breaks work into atomic, checkable tasks
6. You review and approve

**Output:** `docs/specs/{group}/{00X}-{name}/plan.md`

**Gate:** Human approves the plan. Build cannot begin without it.

### `/jim:build`

**Purpose:** Implement the plan using TDD, one task at a time.

**Process:** For each task in `plan.md`:
1. **Red** — Write a failing test (bug: reproduction test; refactor: skip if covered)
2. **Green** — Write minimum code to make it pass
3. **Refactor** — Tidy structural changes (separate from behavioral)
4. Run all tests, confirm green
5. Commit with conventional message
6. Mark task complete, move to next

**Tidy First:** Structural changes (renames, extractions, moves) are never mixed with behavioral changes (new functionality) in the same commit.

**Gate:** All tests pass. `./pre-commit.sh` is green.

### `/jim:review` *(not yet implemented)*

**Purpose:** Quality and security review before PR.

### `/jim:ship` *(not yet implemented)*

**Purpose:** Create PR, deploy, update roadmap status.

---

## Building Jim with Jim

Jim can develop itself through its own SDLC. Skills and agents for the plugin are specs like any other — the "feature" is a plugin component.

```
/jim:spec          → spec out what a skill should do (reads workflow.md)
/jim:plan          → plan the skill's structure and content (optional)
/jim:meta-skill    → build the skill from spec + plan
/jim:meta-agent    → build the agent from spec + plan
```

Jim's own specs live in `docs/specs/jim/` as a group. Jim's strategic docs (`VISION.md`, `ARCHITECTURE.md`, `ROADMAP.md`) live at the plugin root.

---

## Skipping Phases

Not every change needs the full lifecycle:

| Change Type | Start At | Skip |
|-------------|----------|------|
| New feature | `/jim:spec` | — |
| Bug | `/jim:spec` | — |
| Refactor | `/jim:spec` | — |
| Exploratory research | `/jim:research` | Spec, Plan |
| Strategic change | `/jim:vision` or `/jim:arch` | — |
| Hotfix (urgent) | `/jim:build` | Spec, Plan |
| Typo/docs | Direct commit | Everything |

---

## Rules of Engagement

1. **Human-in-the-Loop:** Agents STOP after each phase gate or atomic task. No autonomous multi-phase execution.

2. **Differential Updates:** Every `/jim:` command creates if the artifact doesn't exist, refines based on new context if it does. Never overwrites blindly.

3. **No Ghost Tasks:** If a code change isn't in the plan, update the plan first.

4. **Strategic Alignment:** Specs reference the vision and architecture. The PM checks this during `/jim:spec`.

5. **Verification Loops:** Every code change is verified by test, linter, or type checker.

6. **Progressive Disclosure:** SKILL.md stays under 500 lines. Templates in `assets/`, reference docs in `references/`.
