# Resolve Paths — Shared Preamble

This file is the shared config-resolution preamble invoked by every skill's step 1. It reads `.jim/config.md`, validates it against `skills/_shared/config-schema.md`, and resolves every `{path.*}`, `{specs.*}`, and `{workflow.*}` placeholder so that the invoking skill's subsequent steps substitute resolved values at point of use.

The forcing function is the placeholder syntax itself: skill bodies contain `{path.*}` strings, which are not valid filesystem arguments. To complete any tool call that targets a configurable path, the agent must resolve the placeholder first. There is no externalized table — config adherence comes from the impossibility of passing an unresolved placeholder to a tool, not from emitting an audit artifact every invocation.

`_shared/` is a plugin contract, not an overlayable surface. See the Overlay Boundary section in `skills/_shared/config-schema.md`.

---

## Procedure

Skills invoke this preamble by reading this file and following the steps below. Every step runs in order on every skill invocation — no caching across invocations, no "once per session" shortcuts.

### Step 1 — Read the schema

Read `skills/_shared/config-schema.md`. Parse its frontmatter `keys:` list to obtain the authoritative set of valid keys, their defaults, and their types. Read its **Validation Rules** section for the per-type constraints. These are the two inputs the next steps need.

### Step 2 — Read the project config

Read `.jim/config.md` from the project root if it exists.

- **If the file is absent:** treat every key as present at its schema default. Skip step 3 and go to step 4.
- **If the file exists:** parse its YAML frontmatter. Only the frontmatter is consumed; the prose body of `.jim/config.md` is documentation for the user and is not read by the preamble.

### Step 3 — Validate every key and value

For each key in the parsed `.jim/config.md` frontmatter:

1. **Unknown-key check.** If the key is not listed in the schema's `keys:` frontmatter, halt with the error format below, using `unknown key — not listed in skills/_shared/config-schema.md` as the reason.
2. **Value-type validation.** Apply the schema's Validation Rules for the key's type:
   - `file-path` / `directory-path`: value must be a relative path; must not contain `..` segments after normalization; must not begin with `/`. Normalize the path by following symlinks (realpath semantics) — the final resolved target must lie within the project root. A symlink whose target lies outside the project root is rejected even if the lexical path is clean.
   - `positive-integer`: value must parse as an integer ≥ 1.
   - `boolean`: value must be literal YAML `true` or `false`. Case variants and legacy forms (`yes`, `no`, `on`, `off`, `True`, `FALSE`) are rejected.
   - `string`: any string accepted.
3. **On any violation:** halt with the error format below, naming the offending key and the specific reason from the Validation Rules.

Fallback to defaults on validation failure is prohibited. Validation is strict — every violation surfaces to the user as a hard stop.

### Step 4 — Build the resolved map

For every key declared in the schema:

- If the key appeared in `.jim/config.md` and passed validation: its resolved value is the user's configured value.
- If the key did not appear in `.jim/config.md`: its resolved value is the schema default.

The resolved map is held internally for the rest of the skill invocation. It is not emitted to the conversation — the user has trusted their `.jim/config.md` (if present) and the schema defaults (otherwise); per-invocation visibility of the resolution is unnecessary noise.

### Step 5 — Substitute placeholders at point of use

In the invoking skill's subsequent steps, every occurrence of a `{path.*}`, `{specs.*}`, or `{workflow.*}` placeholder is substituted with its resolved value at the point of use. The resolved map from step 4 is the single source for substitution; do not resolve placeholders ad hoc from memory or by re-reading `.jim/config.md`.

Placeholder strings never flow into tool call arguments unresolved. If the invoking skill's prose contains `Write({path.backlog}, ...)`, the tool call is `Write(BACKLOG.md, ...)` (or the user's override).

---

## Error format

When any validation step halts, emit exactly these three lines, in this order, and stop execution:

```
✗ Config validation failed.
✗ Key: `{offending key name}`
✗ Reason: {human-readable reason from the Validation Rules}
```

Additional rules for the error emission:

- The key name is wrapped in inline backticks. A YAML-legal key that contains markdown metacharacters or control content cannot inject into agent context when rendered inside backticks.
- The raw offending **value** is never echoed in the error text. The reason names the constraint that was violated, not the value that violated it. If an operator needs the raw value for diagnostics, they can read `.jim/config.md` directly.
- After emitting the three lines, stop the current skill invocation. Do not proceed to the invoking skill's subsequent steps. Do not fall back to defaults.

---

## Invariants

- **No silent fallback.** Every validation failure halts; defaults only apply to keys that are absent from `.jim/config.md`, never to keys that are present but invalid.
- **Every invocation.** The preamble runs at the start of every skill invocation, uniformly. There is no "already resolved" shortcut across invocations — subskill calls, repeat calls within a session, and first-time calls all run the resolution.
- **Resolve before tool call.** No `{path.*}`, `{specs.*}`, or `{workflow.*}` placeholder may flow into a tool call unresolved. The placeholder syntax itself is the forcing function — passing an unresolved placeholder to a tool would create a literal-string file or directory, which is impossible to mistake for correct behavior.
- **Schema is the authority.** Any change to the set of valid keys, defaults, or value constraints goes through `skills/_shared/config-schema.md`. This file describes how to *use* the schema; it does not duplicate its contents.
