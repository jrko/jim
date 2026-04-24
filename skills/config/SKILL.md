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

### 1. Resolve config

Follow `skills/_shared/resolve-paths.md` before proceeding. Do not reference any `{path.*}` placeholder until the preamble's resolved-paths table has been emitted.

### 2. Check for existing config

Check if `.jim/config.md` exists at the project root.

- **Exists:** This is an update. Read the file. Summarize current configuration to the user: which keys are set, which are using defaults. Ask: "What do you want to change?"
- **Does not exist:** Fresh creation. Proceed to interview.

### 3. Interview

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

### 4. Generate config

Read `skills/_shared/config-schema.md` for the authoritative keys, defaults, and value constraints. Read `assets/config-template.md` for the user-facing prose scaffolding that accompanies the frontmatter.

Build `.jim/config.md`:
- **Frontmatter:** Only include keys the user explicitly set to non-default values. Validate every non-default value against the schema's Validation Rules before including it; if the user supplied a value that would fail validation, surface the problem before writing rather than shipping a file that will hard-error on next invocation. Empty frontmatter means all defaults.
- **Body:** Copy the prose body from `assets/config-template.md`. The template points the user to `skills/_shared/config-schema.md` as the authoritative reference, so do not inline the full key list into the user's `.jim/config.md`.

### 5. Present and stop

Show the generated config to the user.

- **New creation:** Ask: "Want me to write this, or would you like changes first?"
- **Update:** Show a summary of what changed. Ask: "Want me to apply these changes, or would you like adjustments?"

Create the `.jim/` directory if it doesn't exist. Use Write for new files, Edit for updates. Never auto-apply.

If the user asked for overlay directories, create them as empty directories.
