---
name: vision
description: >
  Create or update the project VISION.md — problem statement, target audience,
  competitive landscape, non-goals, and product north star. Use when the user
  invokes /jim:vision, asks about product direction, or wants to define what
  the project is and isn't. Do not use for technical architecture (/jim:arch),
  execution sequencing (/jim:roadmap), or scoping individual work items (/jim:spec).
agent: pm
---

# /jim:vision

Create or update the project's VISION.md through a section-by-section collaborative interview.

*(The `agent: pm` field in this frontmatter is a jim documentation convention, not a Claude Code routing mechanism.)*

## Process

### 1. Seed the conversation

Use `$ARGUMENTS` as a project name or topic hint. If empty, ask: "What project or product do you want to define a vision for?"

### 2. Read context

Read ARCHITECTURE.md from the project root if it exists — it's a locked constraint. Don't contradict technical decisions already made.

If missing, note conversationally: "No ARCHITECTURE.md yet — you might want to create one after this." Do not block.

### 3. Check for existing VISION.md

Read VISION.md from the project root.

- **Exists:** This is a differential update. Read the content. Tell the user: "I see an existing VISION.md. I'll walk through each section and suggest changes based on our conversation." Identify which sections are well-defined vs. which need work.
- **Does not exist:** Fresh creation. Proceed to interview.

### 4. Section-by-section interview

Walk through the 7 template sections in order. For each section:

1. Explain what the section captures (one sentence)
2. Ask 1-2 bounded questions. Use numbered options where appropriate (3-5 options + "describe your own")
3. Recursive drill-down on vague statements — "for developers" → "which developers? Frontend? Backend? DevOps? Junior? Senior?"
4. Once a clear answer emerges, draft the section content and move to the next

**Section priorities:**

| Priority | Sections | Approach |
|----------|----------|----------|
| Deep interview | Problem Statement, Target Audience, Non-Goals | These are the critical strategic decisions. Spend time here. |
| Moderate interview | Solution Statement, Competitive Landscape | Important but often clearer once problem/audience are defined. |
| Light interview | Product North Star, Roadmap Trajectory | Draft with reasonable defaults from prior answers, refine with user. |

**Context dump mode:** If the user pastes existing docs, PRDs, or notes, skip redundant questions and extract answers from the provided context. Label any assumptions explicitly: "I'm assuming X from your notes — correct?"

### 5. Generate VISION.md

Read `assets/vision-template.md`. Fill each section with interview results. Keep it concise — the goal is clarity of direction, not exhaustive documentation.

Write to `VISION.md` at the project root.

### 6. Silent self-check

Before presenting, validate against these anti-patterns:

- **"For everyone" targeting** — Target Audience is too broad or doesn't exclude anyone. Fix: narrow the audience, add explicit exclusions.
- **Solution-first framing** — Problem Statement contains solution language ("we need to build X"). Fix: reframe around the user pain.
- **Technical creep** — Any section contains implementation details (API specs, DB schemas, architecture). Fix: move to a note suggesting `/jim:arch`.

Auto-correct violations before presenting.

### 7. Present and stop

Show the drafted VISION.md to the user.

- **Differential update:** Show a summary of changes by section before applying. Ask: "Want me to apply these changes, or would you like adjustments?"
- **New creation:** Ask: "Want me to write this, or would you like changes first?"

Use Edit for updates, Write for new files. Never auto-apply.

### 8. Redirect technical specifics

If the user brings up implementation details during the interview, redirect: "That sounds like an architecture decision — want to run `/jim:arch` after we finish the vision?"

VISION.md covers product direction only.
