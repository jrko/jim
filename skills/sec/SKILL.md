---
name: sec
description: >
  Perform security analysis of specs, plans, or arbitrary project files. In
  spec-scoped mode, produces a security.md artifact with actionable findings.
  In ad-hoc mode, delivers analysis directly in conversation. Use when the user
  invokes /jim:sec, asks for a security review, threat model, or wants to check
  a spec or plan for security gaps. Do not use for runtime security scanning,
  post-build code review (/jim:review), or compliance audits.
agent: security
argument-hint: "[spec-dir | file-path | directory]"
---

# /jim:sec

Perform security analysis using a hybrid approach: freeform expert review followed by a STRIDE completeness sweep. Produces actionable findings with severity, suggestions, and routing.

*(The `agent: security` field in this frontmatter is a jim documentation convention, not a Claude Code routing mechanism.)*

## Argument Routing

Use `$ARGUMENTS` to determine the review target and mode:

| Input | Behavior |
| :--- | :--- |
| Empty | Ask: "What should I review? Provide a spec directory, file path, or directory." |
| Path to a directory containing `spec.md` | Spec-scoped mode: review spec and/or plan, produce `security.md` sibling |
| Any other file or directory path | Ad-hoc mode: review the target, deliver findings in conversation |

**Mode detection:** Check if the argument path contains a `spec.md` file (Glob for `{path}/spec.md`). If found → spec-scoped. Otherwise → ad-hoc.

## Process

### 1. Read config

Read `.jim/config.md` from the project root if it exists. Use any configured `path.*` values instead of the default paths in this skill. If the file doesn't exist or a key is omitted, use the defaults shown below.

### 2. Determine mode and read target

**Spec-scoped mode:**
1. Read `spec.md` from the target directory.
2. Read `plan.md` from the target directory if it exists.
3. If neither exists, stop: "No spec.md or plan.md found in [path]."
4. If only one exists, note the absence: "No plan.md found — reviewing spec only" (or vice versa).

**Ad-hoc mode:**
1. Read the target file or Glob the target directory for relevant files.
2. Focus on files likely to have security implications: auth, config, API definitions, data models, infrastructure.

### 3. Read architectural context

Read `ARCHITECTURE.md` (default, configurable via `.jim/config.md`) if it exists. Note existing:
- Trust boundaries and component relationships
- Data flows and storage patterns
- Authentication/authorization patterns
- Security considerations already documented

If absent, note it and proceed without architectural grounding.

Read `VISION.md` (default, configurable via `.jim/config.md`) if it exists — for strategic context only.

### 4. Check for existing security.md (spec-scoped only)

If `security.md` exists in the target directory, this is a differential update:

1. Read the existing `security.md`.
2. Summarize its current state to the user.
3. Ask: "Want me to update the existing review, or start fresh?"
4. If updating, use Edit to preserve sections the user didn't ask to change.

### 5. Freeform expert review

Analyze the target as an experienced security engineer. Focus on:

**Spec-phase lens** (when reviewing spec.md):
- Data classification gaps — PII, credentials, session data without retention/deletion policies
- Missing security requirements — no authZ model, no rate limiting, no input validation specified
- Unaddressed trust boundaries — who can access what, under what conditions
- Privacy and compliance omissions
- Threat surface identification — what attack vectors does this feature introduce

**Plan-phase lens** (when reviewing plan.md):
- Flawed mitigations — JWT without revocation, HTTPS without certificate pinning where needed
- Privilege issues — processes running with excessive permissions
- Trust boundary validation between components
- Crypto choices and key management
- Error handling that leaks information
- Dependency risk

**Ad-hoc lens** (when reviewing code/config/other):
- Apply both lenses contextually based on what the file contains
- Focus on implementation-level security: injection vectors, auth bypasses, secrets exposure, insecure defaults

Look for non-obvious, context-specific issues first. This is the creative, expert-judgment phase.

### 6. STRIDE completeness sweep

After the freeform review, systematically evaluate each STRIDE category:

| Category | Question |
| :--- | :--- |
| **Spoofing** | Can an attacker pretend to be someone/something they're not? |
| **Tampering** | Can data be modified in transit or at rest without detection? |
| **Repudiation** | Can a user deny performing an action? Is there an audit trail? |
| **Information Disclosure** | Can sensitive data leak through errors, logs, or side channels? |
| **Denial of Service** | Can the system be made unavailable through resource exhaustion? |
| **Elevation of Privilege** | Can a user gain permissions they shouldn't have? |

**Calibrate depth to complexity.** Not every category applies to every spec. Mark categories as N/A when they clearly don't apply (e.g., Repudiation for a purely internal refactor). But evaluate — don't skip without consideration.

If the STRIDE sweep surfaces issues not caught in freeform review, add them as findings.

### 7. Spawn researcher if needed

If the analysis identifies questions that require deeper investigation:
- "Is this encryption library still considered safe?"
- "What are the known attack vectors for this auth pattern?"
- "What does the compliance landscape look like for this data type?"

Spawn `@jim:researcher` via the Agent tool with a targeted prompt. Wait for results and incorporate into findings.

### 8. Generate findings

For each issue identified, create a finding with all four required fields:

- **Severity:** Critical (design flaw, will create vulnerability) | Notable (gap to address before build) | Advisory (hardening opportunity)
- **Description:** What the issue is — specific, not vague
- **Suggestion:** Concrete actionable recommendation
- **Route:** Spec (requirements gap) | Plan (design flaw) | Backlog (future hardening)

Order findings by severity — critical first.

### 9. Self-check

First check `.jim/skills/sec/references/security-dod.md` — if it exists, use it instead of the built-in. Read `references/security-dod.md` and validate the review against every applicable checklist item. Fix any gaps before proceeding.

### 10. Generate output

**Spec-scoped mode:**
1. First check `.jim/skills/sec/assets/security-template.md` — if it exists, use it instead of the built-in. Read `assets/security-template.md` for the output structure.
2. Populate all sections from the template.
3. Set frontmatter `spec:` to the relative path of the source spec.
4. Set frontmatter `status:`:
   - `Active` — no critical or notable findings, or all are advisory.
   - `Needs Spec Review` — findings routed to spec require attention.
   - `Needs Plan Review` — findings routed to plan require attention.
5. Set frontmatter `date:` to today's date.
6. Write to `{spec-dir}/security.md` (Write for new, Edit for updates).

**Ad-hoc mode:**
1. Present findings directly in conversation using the same finding structure.
2. Include the STRIDE coverage table.
3. Do not write a file.

### 11. Present and route

Show the findings to the user.

**Spec-scoped routing offer:**
> "Want me to route any of these findings? I can feed them back into the spec (`/jim:spec`), modify the plan (`/jim:plan`), or add to the backlog (`/jim:backlog`)."

**Ad-hoc routing offer:**
> "Want me to add any of these findings to the backlog (`/jim:backlog`)?"

STOP. Wait for human decision. Do not auto-route findings.

## Validation Checklist

Before presenting, confirm:

- [ ] Every finding has severity, description, suggestion, and route
- [ ] Suggestions are concrete and actionable (not vague)
- [ ] Freeform review was performed before STRIDE sweep
- [ ] All six STRIDE categories are evaluated (relevant, not relevant, or N/A)
- [ ] `ARCHITECTURE.md` was read for grounding (or absence noted)
- [ ] No findings duplicate existing `ARCHITECTURE.md` security considerations
- [ ] Correct mode: spec-scoped writes `security.md`; ad-hoc outputs to conversation
- [ ] Differential update used Edit (not Write) when `security.md` already exists
- [ ] Routing offer matches the mode (spec/plan/backlog for spec-scoped; backlog for ad-hoc)
- [ ] `security-dod.md` checklist passed
