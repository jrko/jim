# Resolve Paths — Shared Preamble

Invoked by every skill's step 1. Reads `.jim/config.md`, validates it against `skills/_shared/config-schema.md`, and resolves every `{path.*}`, `{specs.*}`, and `{workflow.*}` placeholder for substitution at point of use in the invoking skill's subsequent steps.

`_shared/` is a plugin contract, not an overlayable surface. See the Overlay Directory section in `skills/_shared/config-schema.md`.

---

## Procedure

Every step runs on every skill invocation — no caching across invocations, no "once per session" shortcuts.

### Step 1 — Read the schema

Read `skills/_shared/config-schema.md`. Parse its frontmatter `keys:` list for valid keys, defaults, and types. Read its **Validation Rules** section for per-type constraints.

### Step 2 — Read the project config

Read `.jim/config.md` from the project root if it exists.

- **If absent:** every key resolves to its schema default. Skip step 3, go to step 4.
- **If present:** parse only the YAML frontmatter; the prose body is not read.

### Step 3 — Validate every key and value

For each key in the parsed `.jim/config.md` frontmatter:

1. **Unknown-key check.** If the key is not in the schema's `keys:` frontmatter, halt with reason `unknown key — not listed in skills/_shared/config-schema.md`.
2. **Value-type validation.** Apply the schema's Validation Rules for the key's type:
   - `file-path` / `directory-path`: relative; no `..` segments after normalization; no leading `/`. Path normalization follows symlinks (realpath semantics) — the final resolved target must lie within the project root. A symlink whose target lies outside the project root is rejected even if the lexical path is clean.
   - `positive-integer`: integer ≥ 1.
   - `boolean`: literal YAML `true` or `false`. Case variants and legacy forms (`yes`, `no`, `on`, `off`, `True`, `FALSE`) are rejected.
   - `string`: any string accepted.
3. **On any violation:** halt with the error format below, naming the offending key and the specific reason.

Fallback to defaults on validation failure is prohibited.

### Step 4 — Build the resolved map

For every key declared in the schema:

- Configured value if the key passed validation in step 3.
- Schema default otherwise.

Hold the map internally; do not emit to the conversation.

**Derived placeholders.** In addition to the schema-declared keys, compute the derived placeholders `{jim_path}` and `{jim_run_hook}`. Documented in `skills/_shared/config-schema.md` under "Derived Placeholders".

`{jim_path}`:

- **Source:** Claude Code's session-level primary working directory — the absolute path Claude Code was invoked in. This is the project root for the rest of the session.
- **Expansion form (multi-token):** `jim_path --root='<absolute-project-root>'`. Helper discovery uses Claude Code's plugin `bin/` PATH convention; the placeholder injects only the project root, not the helper path.
- **Quoting algorithm:** single-quote wrap the project root using the `'\''` escape idiom — replace every embedded `'` with `'\''` and surround the result with single-quotes. The placeholder is the single chokepoint between Claude's environment and any Bash subprocess that uses the helper, so quoting at this layer protects every downstream caller.
- **Halt:** if the project root contains a null byte (cannot be represented as a single-quoted shell token), halt with the same error format as schema validation failures. Other unusual bytes (newlines, control chars) pass through as literal bytes inside the single-quoted string — bash preserves them as-is.

`{jim_run_hook}`:

- **Source:** identical to `{jim_path}` — Claude Code's session-level primary working directory.
- **Expansion form (multi-token):** `jim_run_hook --root='<absolute-project-root>'`. Helper discovery uses Claude Code's plugin `bin/` PATH convention; the placeholder injects only the project root, not the helper path.
- **Quoting algorithm:** identical to `{jim_path}` — single-quote wrap with the `'\''` escape idiom.
- **Halt:** identical to `{jim_path}` — null byte in project root halts with the standard validation-failure error format.

Do **not** parse the skill-invocation system-reminder line (`Base directory for this skill: …`) to derive paths. That line is not a documented stable contract and could change between Claude Code versions. If a future placeholder needs the plugin root (e.g., for invocations not covered by the `bin/` PATH convention), derive it from `${CLAUDE_SKILL_DIR}` — the documented stable Claude Code env var — by stripping the trailing `/skills/<skill-name>` segment.

The resolved map gains entries for any helper that resolves to a multi-token shell command rather than a single value. This is the documented pattern for any derived placeholder whose substitution is a command rather than a path.

### Step 5 — Substitute placeholders at point of use

In the invoking skill's subsequent steps, every `{path.*}`, `{specs.*}`, `{workflow.*}`, `{jim_path}`, or `{jim_run_hook}` placeholder is substituted with its resolved value at the point of use. Do not resolve ad hoc from memory or by re-reading `.jim/config.md`. Placeholders never flow into tool call arguments unresolved — `Write({path.backlog}, ...)` becomes `Write(BACKLOG.md, ...)` (or the override). For `{jim_path}` and `{jim_run_hook}` specifically, substitution produces a multi-token shell command: `$({jim_path} path.X)` becomes `$(jim_path --root='/abs/project/root' path.X)`, and `{jim_run_hook} pre-commit` becomes `jim_run_hook --root='/abs/project/root' pre-commit`, inside Bash invocations.

---

## Error format

When any validation step halts, emit exactly these three lines and stop execution:

```
✗ Config validation failed.
✗ Key: `{offending key name}`
✗ Reason: {human-readable reason from the Validation Rules}
```

- The key name is wrapped in inline backticks.
- The raw offending **value** is never echoed. The reason names the violated constraint, not the value. If diagnostics need the raw value, the operator reads `.jim/config.md` directly.
- After emitting, stop. Do not proceed to subsequent skill steps. Do not fall back to defaults.

---

## Invariants

- **No silent fallback.** Validation failures halt; defaults apply only to keys absent from `.jim/config.md`.
- **Every invocation.** The preamble runs at the start of every skill invocation. No "already resolved" shortcut across subskill calls or repeat calls.
- **Resolve before tool call.** No placeholder may flow into a tool call unresolved.
- **Schema is the authority.** Changes to valid keys, defaults, or value constraints go through `skills/_shared/config-schema.md`.
