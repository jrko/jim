---
name: config
description: >
  Create or update the project's .jim/config.md — artifact path, workflow gates,
  spec ID format, and overlay directory. Use when the user invokes /jim:config,
  wants to customize jim for their project layout, or needs to update existing
  configuration. Do not use for product direction (/jim:vision), technical
  architecture (/jim:arch), or scoping work items (/jim:spec).
agent: meta
argument-hint: ""
---

# /jim:config

Create or update `.jim/config.md` to configure jim for the current project.

*(The `agent: meta` field in this frontmatter is a jim documentation convention, not a Claude Code routing mechanism.)*

## Process

### 1. Check for existing config

Check if `.jim/config.md` exists at the project root.

- **Exists:** This is an update. Read the file. Summarize current configuration to the user: which keys are set, which are using defaults. Ask: "What do you want to change?"
- **Does not exist:** Fresh creation. Proceed to interview.

### 2. Interview

Ask about each configuration area. Keep it light — 1-2 questions per area, skip areas the user doesn't care about.

**Paths:**
- "Where do your strategic docs live? (VISION.md, ARCHITECTURE.md, etc.) Default is the project root."
- "Where should specs, brainstorms, and debug reports go? Default is under `docs/`."

If the user describes a unified namespace (e.g., "everything under `docs/jim/`"), map all paths accordingly without asking about each one individually.

**Spec ID format:**
- "Any preference on spec ID format? Default is 3-digit zero-padded (001, 002). You can change the padding width or add a prefix."

Skip if the user has no opinion — defaults are fine.

**Workflow gates:**
- "Want to enforce any workflow gates? Options:"
  - `require-research` — plan phase stops if research.md is missing (default: off)
  - `require-security` — build phase stops if security.md is missing (default: off)
  - `require-plan-approval` — build phase requires approved plan (default: on)

**Overlay:**
- Mention the overlay directory briefly: "You can also override skill templates and references by placing files in `.jim/skills/{skill}/assets/` or `.jim/skills/{skill}/references/`. Want me to scaffold any overlay directories?"

### 3. Generate config

Read `assets/config-template.md` for the default values and documentation structure.

Build `.jim/config.md`:
- **Frontmatter:** Only include keys the user explicitly set to non-default values. Empty frontmatter means all defaults.
- **Body:** Copy the prose documentation from the template so the user has a reference for available keys and defaults.

### 4. Present and stop

Show the generated config to the user.

- **New creation:** Ask: "Want me to write this, or would you like changes first?"
- **Update:** Show a summary of what changed. Ask: "Want me to apply these changes, or would you like adjustments?"

Create the `.jim/` directory if it doesn't exist. Use Write for new files, Edit for updates. Never auto-apply.

If the user asked for overlay directories, create them as empty directories.
