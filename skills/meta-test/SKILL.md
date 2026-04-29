---
name: meta-test
description: >
  Statically audit jim's own skills and agents for config-adherence
  invariants — preamble invocation, schema value rules, literal default
  filenames in tool-argument positions, and `$({jim_path} <key>)`
  placeholder substitution in Bash invocations. Use when the user invokes
  /jim:meta-test, before committing changes to skills or agents, or to
  verify the audit surface is clean. Do not use for runtime exercise of
  skills, behavioral evaluation, or auditing files outside the consumer
  surface (skills/_shared/, references/, assets/, bin/, root strategic
  docs are out of scope).
agent: meta
argument-hint: ""
---

# /jim:meta-test

Statically audit jim's own skills and agents for the four config-adherence invariants established by specs 012-config-adherence and 013-jim-path-helper. The skill produces an in-conversation pass/fail report; it writes nothing to disk and modifies no files. Findings rely on Claude's judgment over markdown source — anchor examples in each check section calibrate borderline cases.

*(The `agent: meta` field in this frontmatter is a jim documentation convention, not a Claude Code routing mechanism.)*

## Process

### 1. Resolve config

Follow `skills/_shared/resolve-paths.md` before proceeding. Resolve every `{path.*}`, `{specs.*}`, or `{workflow.*}` placeholder before passing it to a tool call.
