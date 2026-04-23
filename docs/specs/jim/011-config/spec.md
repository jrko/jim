---
title: "Project Configuration and Overlay"
type: feature
group: jim
id: "011"
status: approved
origin:
  - docs/brainstorms/20260417-config-file.md
---

# 011 Project Configuration and Overlay

## Overview

Add a `.jim/config.md` file for project-level configuration (paths, workflow gates, spec ID format) and a `.jim/` overlay directory for custom skill assets and references, enabling users to adapt jim to their project conventions without forking.

## Problem Statement

Jim hardcodes artifact paths, workflow behavior, and templates across 13 skills and 6 agents. Users with existing project conventions — or even forks of the upstream project with different layouts — must modify skill files directly to change where artifacts are written or which templates are used. This creates maintenance burden when merging upstream changes and prevents adoption by teams with established directory structures.

## User Stories

- As a developer adopting jim into an existing project, I can configure artifact paths so that jim writes specs, strategic docs, and reports to locations that match my project layout.
- As a team lead, I can enforce workflow gates (require research before planning, require security review before building) so that the team follows a consistent SDLC process.
- As a developer, I can override a skill's template by placing a custom version in `.jim/` so that generated artifacts match my team's conventions without modifying the plugin.
- As a developer, I can run `/jim:config` to scaffold or update my project configuration interactively.

## Acceptance Criteria

- [ ] Skills read `.jim/config.md` during their existing "read strategic context" step and use configured values instead of hardcoded defaults
- [ ] When `.jim/config.md` does not exist, all skills behave identically to current behavior (zero-config backward compatibility)
- [ ] Path configuration supports both individual file paths (e.g., `vision: VISION.md`) and directory paths (e.g., `specs: docs/specs`)
- [ ] Workflow gate `require-research: true` causes the plan skill to stop and inform the user when `research.md` is missing for the target spec
- [ ] Workflow gate `require-security: true` causes the build skill to stop and inform the user when `security.md` is missing for the target spec
- [ ] Workflow gate `require-plan-approval: false` allows the build skill to proceed from a draft plan without requiring `status: approved`
- [ ] Spec ID format is configurable via `id-padding` and `id-prefix`, and the spec skill uses these when assigning new IDs
- [ ] When a skill reads an asset or reference file, it checks `.jim/skills/{skill}/assets/{file}` or `.jim/skills/{skill}/references/{file}` first, falling back to the built-in plugin path
- [ ] The overlay applies only to assets and references, not to SKILL.md or agent definitions
- [ ] `/jim:config` creates `.jim/config.md` with empty frontmatter and defaults documented in the prose body when no config exists (scaffolding mode). Skills only read frontmatter — omitted keys use hardcoded defaults.
- [ ] `/jim:config` reads and presents current config for modification when `.jim/config.md` already exists (update mode)
- [ ] `/jim:config` skill is added to the meta agent's `skills:` frontmatter
- [ ] All configured paths are relative to project root
- [ ] Workflow gate defaults match current behavior: `require-research: false`, `require-security: false`, `require-plan-approval: true`

## Out of Scope

- Integration settings (issue trackers, CI/CD, deployment environments) — better suited to CLAUDE.md
- Per-spec-type workflow rules (e.g., "research required for features but not bugs") — simple checklist only
- SKILL.md overlay — plugin skills are namespaced; self-replacement is architecturally awkward
- Agent overlay — Claude Code natively supports project-level agent overrides via `.claude/agents/`
- Effort and model configuration — these are platform-level settings (skill/agent frontmatter, Claude Code CLI); cannot be self-applied by reading a config file at runtime
- Git branch naming, language/stack hints, research scope exclusions — better suited to CLAUDE.md
- Config file validation or schema enforcement — agents read YAML frontmatter natively
- Dynamic config (environment variables, CLI flags overriding config values)

## Open Questions

- [x] ~Should the `/jim:config` skill live on the PM agent or a new lightweight agent?~ → Meta agent. Config scaffolding is plugin infrastructure, same category as `/jim:meta-skill` and `/jim:meta-agent`.
- [x] ~Should config support a `default-group` key for spec grouping?~ → No. The PM agent reliably infers the correct group from context without configuration.
- [x] ~How should effort/model config interact with agent overrides via `.claude/agents/`?~ → Not jim's concern. `.claude/agents/` overrides are Claude Code-native; if a user overrides an agent there, they own its configuration. Jim's config only applies to built-in jim agents.
