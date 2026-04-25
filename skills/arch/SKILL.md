---
name: arch
description: >
  Generate or update ARCHITECTURE.md from actual codebase analysis. Use when
  the user invokes /jim:arch, when starting a new project and needing an
  architecture baseline, or when the architecture document has drifted from
  the real codebase. Do not use for planning specific features (/jim:plan) or
  for implementation (/jim:build).
agent: architect
argument-hint: "[directory-path]"
---

# /jim:arch

Generate or update the architecture doc at `{path.architecture}` from actual codebase scanning. The document reflects reality, not aspirations.

## Argument Routing

Use `$ARGUMENTS` to determine scope:

| Input | Behavior |
| :--- | :--- |
| Empty | Create or update `{path.architecture}` at the project root |
| Directory path | Create or update an architecture doc inside that directory (uses the configured filename) |

## Process

### 1. Resolve config

Follow `skills/_shared/resolve-paths.md` before proceeding. Resolve every `{path.*}`, `{specs.*}`, or `{workflow.*}` placeholder before passing it to a tool call.

### 2. Establish scope

Determine the target path from `$ARGUMENTS`. If empty, the target file is `{path.architecture}` at the project root. If a directory path was provided, the target file is `{directory}/{path.architecture}` — the `{directory}` placeholder is substituted from `$ARGUMENTS` (runtime substitution, no dot in the name), while `{path.architecture}` is resolved from config by the preamble. Naming convention distinguishes the two: runtime placeholders contain no dot; config placeholders always do.

### 3. Read the vision as source context

Check for `{path.vision}` at the project root.

- **Exists:** Read it fully. The architecture serves the vision — where there is tension between the actual code and the stated vision, flag it rather than silently encoding the discrepancy into the architecture document.
- **Missing:** Proceed without it. Note its absence in the Overview if you generate a new file.

### 4. Check for existing architecture doc

Look for an existing architecture doc at the target path.

- **Exists:** This is a differential update. Read the existing document fully. Summarize proposed changes to the user — which sections will be updated, which will be preserved — before writing anything. Use Edit, not Write.
- **Missing:** Generate a new document from `assets/architecture-template.md` (first check `.jim/skills/arch/assets/architecture-template.md` — if it exists, use it instead of the built-in).

### 5. Scan the codebase

Do not fill the template from assumptions. Read actual code.

Use Glob and Grep to populate each section:

- **Project Structure:** Glob `{directory}/**` to map the directory tree. Focus on the top 2-3 levels; annotate directories whose purpose is non-obvious.
- **Core Components:** Grep for entry points, exported functions, class/interface definitions, and module boundaries. Identify what each component exposes and depends on.
- **Data Stores:** Grep for database connections, file reads/writes, cache calls, or persistence patterns. Look for config files that declare data locations.
- **External Integrations:** Grep for HTTP clients, API calls, SDK imports, and third-party service references. Check package manifests (package.json, go.mod, requirements.txt, Cargo.toml) for external dependencies.
- **Deployment & Infrastructure:** Look for Dockerfile, docker-compose.yml, .github/workflows/, Makefile, build scripts. Check package.json `scripts` or equivalent.
- **Security Considerations:** Grep for auth patterns, secret/credential handling, environment variable reads, file permission checks.
- **Development & Testing:** Find test files and directories, CI config, linting config. Identify the test framework and test command.

For each finding, record the file path and relevant line range. The architecture document is grounded in evidence, not inference.

### 6. Generate or update the document

First check `.jim/skills/arch/assets/architecture-template.md` — if it exists, use it instead of the built-in. Read `assets/architecture-template.md` for the section structure.

Fill each section from scan findings:

- Use actual directory names, file paths, and component names from the codebase.
- Write the High-Level System Diagram as a Mermaid flowchart. Use actual component names — not generic "Component A" placeholders.
- If a section has no findings (e.g., no external integrations), write "*None identified.*" rather than removing the section. This signals completeness, not omission.
- If `{path.vision}` flagged a tension between vision and implementation, note it in Security Considerations or a brief "Architecture Notes" at the end.

### 7. Present and stop

Show the completed document (or summarize changes for a differential update). List which sections changed and which sections had no findings.

Ask: "Does this look accurate? Any sections to refine?"

Do not proceed to the next phase.

## Validation Checklist

Before presenting, confirm:

- [ ] Every section is populated from actual code, not from the template placeholder text
- [ ] No generic placeholder names remain (e.g., "Component A", "{project-root}")
- [ ] High-Level System Diagram uses real component names
- [ ] Sections with no findings say "*None identified.*" rather than being removed
- [ ] `{path.vision}` was checked and any tensions are noted
- [ ] Differential update used Edit, not Write
- [ ] File paths in Component sections include actual line-range anchors where relevant
