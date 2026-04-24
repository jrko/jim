---
# Add configuration overrides below. Omitted keys use the schema defaults.
# Skills only read frontmatter — the prose body below is for human reference.
---

# Jim Project Configuration

This file configures jim for your project. Only add keys you want to override — omitted keys use the defaults defined in the jim plugin's schema.

## Authoritative schema

The full list of valid keys, their defaults, and their value constraints lives in the jim plugin at `skills/_shared/config-schema.md`. That file is the single source of truth. It documents:

- every key (`path.*`, `specs.*`, `workflow.*`)
- each key's default value and expected value type
- per-type validation rules (e.g., `path.*` values must be relative, inside the project root, with no `..` or leading `/`)
- which files and directories are overlayable via `.jim/` and which are not

Open `skills/_shared/config-schema.md` in your jim installation to see the current key set. The schema is the authority for what you may write in the frontmatter above.

## Overlay directory

Place custom assets and references under `.jim/` to override built-in plugin files. Skills check the overlay path first, then fall back to the plugin file.

```
.jim/
  config.md                              # this file
  skills/
    spec/
      assets/
        spec-template.md                 # overrides built-in spec template
    build/
      references/
        tdd-guide.md                     # overrides built-in TDD guide
```

Only `assets/` and `references/` under a named skill are overlayable. `SKILL.md`, agent definitions, and everything under `skills/_shared/` are plugin contract and cannot be overridden via `.jim/`. For agent overrides, use Claude Code's native `.claude/agents/` mechanism.
