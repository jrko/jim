# Research: SDLC Researcher Agents and Skills

- **Source Spec:** [docs/jim/specs/004-researcher/research.md]
- **Status:** Complete
- **Date:** 2026-03-12

## 1. Executive Summary
This research explores state-of-the-art "Researcher" agents within the Claude Code and AI-agentic SDLC ecosystem. Unlike academic researchers, these agents focus on **codebase archaeology**, **implementation discovery**, and **technical constraint gathering**. The findings highlight a trend toward "Tiered Research" (Local vs. Web) and "Anchor Identification" as the gold standards for informing the Planning phase.

---

## ## Tier 1: Study These Closely
These frameworks share Jim's philosophy of spec-driven development and provide the most direct patterns for `v1-researcher`.

### **Repo**: [akaszubski/autonomous-dev](https://github.com/akaszubski/autonomous-dev)
**Relevant to**: `/jim:spec`, `v1-researcher`, `v1-architect`

| File | What It Is | Why It Matters for Jim |
| :--- | :--- | :--- |
| [`agents/researcher-local.md`](https://github.com/akaszubski/autonomous-dev/blob/main/agents/researcher-local.md) | A specialized agent for codebase-only context gathering. | Implements a "Local-First" rule to prevent token bloat and hallucination by forcing Grep/Glob use before Web search. |
| [`agents/researcher-web.md`](https://github.com/akaszubski/autonomous-dev/blob/main/agents/researcher-web.md) | A companion agent focused on external documentation and API discovery. | Perfectly maps to Jim's `WebFetch`/`WebSearch` guardrails, separating internal patterns from external knowledge. |

**Key takeaways for Jim:**
* **Model Tiering:** Uses Claude 3.5 Haiku for local research (cost-effective) and Sonnet for web-based synthesis. 
* **Alignment Command:** Includes a `/align` command that validates findings against a `PROJECT.md`, ensuring research doesn't drift from core goals.

### **Repo**: [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done)
**Relevant to**: `v1-researcher`, `v1-architect`

| File | What It Is | Why It Matters for Jim |
| :--- | :--- | :--- |
| [`agents/researcher.md`](https://github.com/gsd-build/get-shit-done/blob/main/agents/researcher.md) | A core orchestrator sub-agent for investigation. | Specifically investigates "implementation approaches" rather than just finding files, providing options for the Planner. |

**Key takeaways for Jim:**
* **Human-in-the-loop Gates:** Like Jim, GSD pauses after the Research/Plan phase for verification, preventing autonomous "runaway" implementation.
* **Atomic Research:** Focuses on researching a single "Phase" at a time, keeping the `research.md` extremely focused and under the word budget.

---

## ## Tier 2: Study for Specific Patterns
These repos offer unique tactical patterns (e.g., specific prompts or multi-platform compatibility) that can enhance `v1-researcher-skill`.

### **Repo**: [nguyenvanduocit/research-kit](https://github.com/nguyenvanduocit/research-kit)
**Relevant to**: `v1-researcher-skill`

| File | What It Is | Why It Matters for Jim |
| :--- | :--- | :--- |
| [`templates/research.md`](https://github.com/nguyenvanduocit/research-kit/blob/main/templates/research.md) | A 10-phase research template. | Provides a structured "Methodology" section that defines *how* the search was conducted (e.g., source evaluation criteria). |

**Key takeaways for Jim:**
* **Principle-Based Research:** Introduces a `/research.principles` command to set the "bar" for evidence (e.g., requiring official docs over blog posts).
* **Target Audience Focus:** Research results are tailored for the "Target Audience" (e.g., engineering leadership), which Jim can use to differentiate between Architect and Coder needs.

### **Repo**: [SuperClaude-Org/SuperClaude_Framework](https://github.com/SuperClaude-Org/SuperClaude_Framework)
**Relevant to**: `v1-researcher`, `v1-architect`

| File | What It Is | Why It Matters for Jim |
| :--- | :--- | :--- |
| [`personas/technical-analyst.md`](https://github.com/SuperClaude-Org/SuperClaude_Framework/blob/main/personas/technical-analyst.md) | A "Cognitive Persona" for technical deep-dives. | Uses "Reasoning Loops" to double-check anchors and "Deep Research Mode" for citation chains. |

**Key takeaways for Jim:**
* **Orchestration Mode:** Coordinates Web search, code analysis, and documentation lookup automatically, preventing the agent from getting stuck in one tool.
* **Token-Efficiency Mode:** Explicitly reduces context consumption by 30-50%, aligning with Jim's "20-Line Rule."

---

## ## Tier 3: Reference Only
Useful for edge cases or non-core SDLC research tasks.

### **Repo**: [github/spec-kit](https://github.com/github/spec-kit)
**Relevant to**: `v1-researcher`

| File | What It Is | Why It Matters for Jim |
| :--- | :--- | :--- |
| [`templates/plan.md`](https://github.com/github/spec-kit/blob/main/templates/plan.md) | A technical implementation plan template. | Includes a `research` section in the plan itself, ensuring findings are never detached from the implementation roadmap. |

### **Repo**: [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents)
**Relevant to**: `v1-researcher`

| File | What It Is | Why It Matters for Jim |
| :--- | :--- | :--- |
| [`08-business-product/ux-researcher.md`](https://github.com/VoltAgent/awesome-claude-code-subagents/blob/main/categories/08-business-product/ux-researcher.md) | A researcher for user needs and personas. | While not for code, the "Verification Checklist" at the bottom is a masterclass in Definition of Done (DoD). |

**Key takeaways for Jim:**
* **DoD Checklists:** The strict checklist format (e.g., "Bias minimized systematically") is highly effective for ensuring the agent doesn't stop until all anchors are documented.

---

## 3. Risks & Recommendations
- **Risk:** Most frameworks suffer from "Token Sprawl" in research. Jim’s "20-Line Rule" and word budget are already state-of-the-art for keeping costs low.
- **Recommendation:** Implement a "Local vs. Web" split in the `v1-researcher` tool-use instructions. Force a `Grep`/`Glob` pass of the codebase *before* allowing `WebSearch` to prevent the agent from suggesting generic solutions over project-specific patterns.
- **Recommendation:** Adopt the "Anchor Summary" pattern from `autonomous-dev`, where the agent must explain *why* a file is an anchor (e.g., "Contains the main routing logic for this module").