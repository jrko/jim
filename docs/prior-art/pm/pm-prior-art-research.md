# Prior Art Study Guide for Jim Plugin PM Agent & Skills

## Tier 1: Study These Closely

### 1. deanpeters/Product-Manager-Skills

**Repo:** https://github.com/deanpeters/Product-Manager-Skills

**Relevant to:** `/jim:vision`, `/jim:brainstorm`, `/jim:roadmap`, `@jim:pm` agent persona

| File | What It Is | Why It Matters for Jim |
|------|-----------|----------------------|
| [skills/positioning-workshop/SKILL.md](https://github.com/deanpeters/Product-Manager-Skills/blob/main/skills/positioning-workshop/SKILL.md) | Interactive skill that walks a user through Geoffrey Moore positioning discovery via adaptive questions → structured output | Direct prior art for `/jim:vision`. Jim's vision skill needs to extract problem statement, value prop, target audience, competitive landscape — this skill does exactly that. The "ask 3-5 questions → synthesize → output artifact" pattern maps to Jim's "PM asks clarifying questions (1-2 at a time)" pattern. |
| [skills/product-strategy-session/](https://github.com/deanpeters/Product-Manager-Skills/tree/main/skills) | Workflow skill orchestrating positioning → problem statement → JTBD → roadmap over 2-4 weeks | Shows how to chain component skills into a multi-step strategic process. Jim's `/jim:vision` is simpler (single output) but the questioning sequence and orchestration pattern is directly reusable. |
| [skills/roadmap-planning/](https://github.com/deanpeters/Product-Manager-Skills/tree/main/skills) | Epics → prioritization (RICE/ICE/Kano) → sequencing workflow (1-2 weeks) | Direct prior art for `/jim:roadmap`. Jim needs to help users prioritize and sequence milestones — this skill encodes the frameworks for that. |
| [skills/opportunity-solution-tree/](https://github.com/deanpeters/Product-Manager-Skills/tree/main/skills) | Teresa Torres' OST framework for mapping opportunities to solutions | Model for how `/jim:brainstorm` could offer optional structure beyond freeform ideation. |
| [skills/jobs-to-be-done/](https://github.com/deanpeters/Product-Manager-Skills/tree/main/skills) | JTBD framework as a component skill | Reusable strategic lens for `/jim:vision` — understanding what job the product does for the user. |
| [skills/problem-statement/](https://github.com/deanpeters/Product-Manager-Skills/tree/main/skills) | Problem statement template with quality criteria | Template pattern for Jim's `vision-template.md` — shows how to structure an artifact template with Purpose, Key Concepts, Application, Examples, Common Pitfalls. |
| [skills/prd-development/](https://github.com/deanpeters/Product-Manager-Skills/tree/main/skills) | PRD workflow: problem → solution → stories (2-4 days) | Prior art for `/jim:spec` — shows how a PM skill transforms fuzzy requirements into a structured deliverable. |
| [PLANS.md](https://github.com/deanpeters/Product-Manager-Skills/blob/main/PLANS.md) | Skill dependency graph showing how all skills connect | Architecture reference for how Jim's PM skills should reference each other (e.g., vision feeds into roadmap feeds into spec). |
| [CLAUDE.md](https://github.com/deanpeters/Product-Manager-Skills/blob/main/CLAUDE.md) | Skill template standard and design philosophy | Defines the three-tier skill typing (component/interactive/workflow) and the standard structure every skill follows. Reference for Jim's `skill-standards.md`. |

**Key takeaways for Jim:**
- The three skill types (component template, interactive advisor, workflow) are a proven taxonomy. Jim's skills are all workflows or interactive — consider whether any should be component templates.
- Every skill includes explicit anti-patterns ("don'ts") that prevent generic AI output. Jim's PM agent persona should encode similar guardrails.
- Skills produce drafts the user refines, not finished artifacts. Aligns with Jim's "Human approves" gate pattern.

---

### 2. automazeio/ccpm

**Repo:** https://github.com/automazeio/ccpm

**Relevant to:** `/jim:spec`, `/jim:plan`, traceability enforcement, artifact directory structure

| File | What It Is | Why It Matters for Jim |
|------|-----------|----------------------|
| [.claude/commands/pm/](https://github.com/automazeio/ccpm/tree/main/.claude/commands/pm) | 30+ PM command definitions including `prd-new`, `prd-parse`, `epic-decompose`, `epic-oneshot` | Closest structural analog to Jim's spec→plan pipeline. `prd-new` (guided brainstorming → PRD) maps to `/jim:spec`. `prd-parse` (PRD → technical epic) maps to the `/jim:spec` → `/jim:plan` handoff. Study the command markdown files for how they structure prompts and context injection. |
| [.claude/epics/{epic-name}/](https://github.com/automazeio/ccpm) | Local workspace: `epic.md` (implementation plan) + `001.md`, `002.md` (individual tasks) + `updates/` (WIP tracking) | Direct prior art for Jim's `docs/specs/{group}/{00X}-{name}/` directory with `spec.md` and `plan.md`. Same pattern of co-locating related artifacts. Study how tasks are numbered and how the `updates/` subdirectory tracks progress. |
| [.claude/prds/](https://github.com/automazeio/ccpm) | PRD storage directory | Analog to Jim's `docs/specs/` as the canonical location for product specs. |
| [.claude/rules/](https://github.com/automazeio/ccpm) | Reusable rule files referenced by commands | Pattern for whether Jim's `references/` subdirectories should store shared rules separately from skill instructions. |
| [.claude/agents/](https://github.com/automazeio/ccpm) | Task-oriented agents (code-analyzer, file-analyzer, test-runner, parallel-worker) | Agent definitions for context preservation. Compare to Jim's `jim/agents/` structure. |
| [COMMANDS.md](https://github.com/automazeio/ccpm/blob/main/COMMANDS.md) | Full command reference including context management (`/context:create`, `/context:update`, `/context:prime`) | The context priming pattern — loading project state before a work session — is relevant to how Jim's commands should read VISION.md and ARCHITECTURE.md at the start of `/jim:spec`. |

**Key takeaways for Jim:**
- CCPM's "No Vibe Coding" rule (every line of code traces to a spec) is the same principle as Jim's "No Ghost Tasks." Study how CCPM enforces it mechanically through the `PRD → Epic → Task → Issue → Code → Commit` chain.
- CCPM operates local-first, syncing to GitHub only when explicitly told. Jim's artifacts are also local-first. This validates the approach.
- CCPM's pipeline is linear (PRD → Epic → Tasks → Execute). Jim's feedback loop (build discovers spec gap → update spec → update plan) is a differentiator — CCPM doesn't have upstream correction.
- **Ignore** CCPM's parallel agent execution, Git worktree management, and GitHub Issues sync — Jim is human-in-the-loop with sequential tasks, not parallel agents.

---

### 3. gsd-build/get-shit-done (GSD)

**Repo:** https://github.com/gsd-build/get-shit-done

**Relevant to:** `/jim:spec` (clarifying questions pattern), `/jim:roadmap`, artifact structure, differential updates

| File | What It Is | Why It Matters for Jim |
|------|-----------|----------------------|
| [agents/gsd-planner.md](https://github.com/gsd-build/get-shit-done/blob/main/agents/gsd-planner.md) | Planner agent that consumes upstream context (CONTEXT.md from discuss-phase, RESEARCH.md) and decomposes phases into tasks | Key pattern: treats user decisions from discuss-phase as **locked decisions it will not revisit**. Jim's `@jim:pm` needs the same behavior — when `/jim:spec` reads VISION.md, those strategic decisions are constraints, not suggestions. Also study the two-step context assembly (digest for selection, full read for understanding). |
| [commands/gsd/](https://github.com/gsd-build/get-shit-done/tree/main/commands/gsd) | Command definitions for `new-project`, `discuss-phase`, `define-requirements`, `research-project`, `plan-phase` | `discuss-phase` was redesigned with "intelligent gray area analysis" — identifies which aspects of a phase are open for discussion (UI, UX, behavior), lets user multi-select, then deep-dives each. This pattern directly informs how `/jim:spec`'s "PM asks clarifying questions" step should work: analyze the rough idea → identify which dimensions need clarification → let the user steer. |
| [docs/USER-GUIDE.md](https://github.com/gsd-build/get-shit-done/blob/main/docs/USER-GUIDE.md) | Full documentation including `.planning/` artifact structure | `.planning/` contains `PROJECT.md` (vision, always loaded), `REQUIREMENTS.md` (scoped requirements with IDs), `ROADMAP.md` (phases with status), `STATE.md` (decisions, blockers, session memory). Compare directly to Jim's VISION.md + ROADMAP.md + spec directory layout. `STATE.md` (session memory) is something Jim doesn't have — consider whether the PM agent needs persistent session context. |
| [CHANGELOG.md](https://github.com/glittercowboy/get-shit-done/blob/main/CHANGELOG.md) | Documents the evolution of discuss-phase, requirement traceability, and planning pipeline | Shows how the `new-project → research-project → define-requirements → create-roadmap` pipeline was built iteratively. The requirements traceability feature (phases map to requirement IDs with 100% coverage validation) is prior art for Jim's "every spec traces to vision and architecture" rule. |

**Key takeaways for Jim:**
- GSD's discuss-phase gray-area analysis is the best prior art for how `/jim:spec` should handle clarifying questions — not generic questions, but targeted questions about specific uncertain dimensions.
- The locked-decisions pattern (upstream decisions are constraints, not suggestions) should be a core behavior of `@jim:pm`.
- GSD's size discipline (plans must complete within ~50% of context window; 2-3 tasks max per plan) is a practical guardrail Jim should adopt for plan.md.
- GSD's `--auto @prd.md` ability to ingest an existing document is worth considering for `/jim:spec` — users may already have a rough spec or PRD to import.
- **Ignore** GSD's execution layer (parallel waves, subagent spawning, model profiles, auto-advance) — Jim's build phase works differently.

---

## Tier 2: Study for Specific Patterns

### 4. SuperClaude Framework — `pm_agent/` Module

**Repo:** https://github.com/SuperClaude-Org/SuperClaude_Framework

**Relevant to:** `@jim:pm` agent behavioral patterns, future `/jim:review` phase

| File | What It Is | Why It Matters for Jim |
|------|-----------|----------------------|
| [src/superclaude/pm_agent/confidence.py](https://github.com/SuperClaude-Org/SuperClaude_Framework/tree/master/src/superclaude/pm_agent) | Pre-execution confidence check: ≥90% proceed, 70-89% present alternatives, <70% ask questions | Pattern for `@jim:pm` behavior during `/jim:spec`. When the PM agent isn't confident about spec type, scope, or alignment with VISION.md, it should ask more questions rather than guess. This gives a concrete mechanism for "how uncertain is too uncertain to proceed." |
| [src/superclaude/pm_agent/reflexion.py](https://github.com/SuperClaude-Org/SuperClaude_Framework/tree/master/src/superclaude/pm_agent) | Error learning — documents successful patterns and analyzes mistakes for future sessions | Prior art for Jim's future `/jim:review` and `/jim:ship` phases. When a build reveals spec gaps (triggering the feedback loop), the PM agent could log what went wrong to improve future specs. |
| [src/superclaude/pm_agent/self_check.py](https://github.com/SuperClaude-Org/SuperClaude_Framework/tree/master/src/superclaude/pm_agent) | Post-implementation validation — checks if output matches intent | Pattern for the human approval gate in Jim's workflow. Before presenting a spec to the user, the PM agent could self-check: does this spec actually address the user's stated idea? Does it align with VISION.md? |
| [src/superclaude/commands/brainstorm.md](https://github.com/SuperClaude-Org/SuperClaude_Framework/blob/master/src/superclaude/commands/brainstorm.md) | Socratic requirements discovery with multiple perspectives | Pattern for `/jim:brainstorm` — how to structure multi-perspective ideation beyond freeform notes. |
| [src/superclaude/commands/spec-panel.md](https://github.com/SuperClaude-Org/SuperClaude_Framework/blob/master/src/superclaude/commands/spec-panel.md) | Multi-expert specification review — simulates multiple expert perspectives reviewing a spec | Consider whether `/jim:spec` should have a self-review step where the PM agent checks the spec from different angles (user perspective, technical feasibility, scope creep risk) before presenting it to the human. |

**Key takeaways for Jim:**
- The confidence threshold is a simple, adoptable pattern that makes agent behavior more predictable.
- Self-check before presenting output to the human aligns with Jim's "agents STOP and wait for approval" philosophy — the agent should be confident before stopping.
- **Ignore** SuperClaude's always-on PM orchestrator pattern, behavioral routing/auto-activation, 16-agent army, MCP server dependencies, and the plugin system that doesn't exist yet. Jim's explicit-command philosophy is deliberately different from auto-activation.

---

### 5. Pimzino/claude-code-spec-workflow

**Repo:** https://github.com/Pimzino/claude-code-spec-workflow

**Relevant to:** `/jim:vision` (steering documents), `/jim:spec` (spec lifecycle)

| File | What It Is | Why It Matters for Jim |
|------|-----------|----------------------|
| [/spec-steering-setup](https://github.com/Pimzino/claude-code-spec-workflow) | Creates three steering documents: `product.md`, `tech.md`, `structure.md` | Jim splits steering into VISION.md (product) and ARCHITECTURE.md (technical). Pimzino splits it three ways. Compare the decomposition — does Jim need a third steering doc for project structure/conventions, or is ARCHITECTURE.md sufficient? |
| [/spec-create and /spec-execute](https://github.com/Pimzino/claude-code-spec-workflow) | Spec creation → task execution with validation at each gate | The recommendation to use **Opus for spec creation** (`/spec-create`) and **Sonnet for implementation** (`/spec-execute`) is a practical insight. Jim could adopt this: `@jim:pm` (spec, vision, roadmap) benefits from the strongest reasoning model, while `@jim:coder` (build) may work fine with a faster model. |
| [Core workflow: spec-task-executor, spec-requirements-validator, spec-design-validator](https://github.com/Pimzino/claude-code-spec-workflow) | Validation agents that check specs and designs before execution proceeds | Prior art for how Jim's phase gates could include automated validation, not just human approval. A spec-validator could check: are acceptance criteria testable? Does the spec reference VISION.md? Is scope bounded? |

**Key takeaways for Jim:**
- The three-document steering split is worth comparing to Jim's two-document split (VISION.md + ARCHITECTURE.md).
- Model selection per agent role is a practical optimization.
- **Ignore** the MCP version migration, the dashboard/tunnel features, and the npm package distribution — these are infrastructure concerns, not PM patterns.

---

## Tier 3: Reference Only

### 6. phuryn/pm-skills

**Repo:** https://github.com/phuryn/pm-skills

100+ PM skills organized as Claude Code plugins covering product strategy, discovery, market research, analytics, marketing/growth, go-to-market, and execution. Curated by Paweł Huryn (The Product Compass). Useful for browsing the breadth of PM skill categories to spot gaps in Jim's PM agent coverage or to find naming conventions. Individual skill depth is less documented than deanpeters. The plugin marketplace architecture is not relevant to Jim.

### 7. VoltAgent/awesome-claude-code-subagents — Product Manager Subagent

**Repo:** https://github.com/VoltAgent/awesome-claude-code-subagents

**File:** [categories/08-business-product/product-manager.md](https://github.com/VoltAgent/awesome-claude-code-subagents/blob/main/categories/08-business-product/product-manager.md)

Single-file PM subagent prompt defining a "senior product manager" persona covering product strategy, user research, feature prioritization, and go-to-market execution. Useful as a starting reference for `jim/agents/pm.md`'s persona section. Too generic for Jim's structured approach — no workflow, no state management, no skill composition.

### 8. github/spec-kit

**Repo:** https://github.com/github/spec-kit

GitHub's official spec-driven development toolkit. Defines the `constitution → specify → plan → tasks → implement` pipeline. Worth reading for the overall philosophy (intent as source of truth, specs as executable artifacts) and the concept of a "constitution" (analogous to Jim's VISION.md as the foundational governing document). Their tooling is agent-agnostic and less opinionated than Jim — useful as methodology reference, not implementation reference.

### 9. gotalab/cc-sdd

**Repo:** https://github.com/gotalab/cc-sdd

Spec-driven development supporting 8 agents × 13 languages. Implements `steering → spec-init → spec-design → spec-tasks → spec-impl` pipeline with customizable templates. Worth monitoring for their template customization approach (customize once, all agents use consistent output). Less relevant than GSD or CCPM because it's more generic and doesn't have PM-specific strategic tooling.