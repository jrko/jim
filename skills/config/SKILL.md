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

Follow `skills/_shared/resolve-paths.md` before proceeding. Resolve every `{path.*}`, `{specs.*}`, or `{workflow.*}` placeholder before passing it to a tool call.

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

Read `skills/_shared/config-schema.md` for the authoritative keys, defaults, and value constraints. Read `assets/config-template.md` for the scaffolding.

Build `.jim/config.md`:

- Use `assets/config-template.md` verbatim. Insert the user's overrides as keys inside the frontmatter block.
- Only include keys the user explicitly set to non-default values. Validate every value against the schema's Validation Rules; surface any violation before writing.
- Empty frontmatter (no overrides) is valid — write the template as-is.
- Do not add a prose body. The schema and overlay docs live in `skills/_shared/config-schema.md`.

### 5. Present and stop

Show the generated config to the user.

- **New creation:** Ask: "Want me to write this, or would you like changes first?"
- **Update:** Show only the frontmatter changes (added keys, removed keys, changed values). Ask: "Want me to apply these changes, or would you like adjustments?"

Create the `.jim/` directory if it doesn't exist. Use Write for new files, Edit for updates. Never auto-apply.

If the user asked for overlay directories, create them as empty directories.
